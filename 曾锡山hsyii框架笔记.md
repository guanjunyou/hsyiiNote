## 曾锡山HsYii框架笔记

### 需求一：在ClubNews页面上取出PayType对应的数据库(pay_type)里的缴费类型进行遍历填写并结合ClubNews的项目类型一起传到ClubPaySet对应的数据库(hash)

![]()

**要求实现：**

**1，实施输入实时显示总钱数，并在下一次进入该页面时输入框内数字不发生变化**

**2，pay_type里面有多少个类型在ClubNews/update.php就要显示多少个输入框**

![image-20211213093400146](https://s3.bmp.ovh/imgs/2021/12/349bdc52fa522b6a.jpg)

![image-20211213093427149](https://s3.bmp.ovh/imgs/2021/12/8833883fafa8c1ca.jpg)

![image-20211213093459965](https://s3.bmp.ovh/imgs/2021/12/8ae61a277c1a6dde.jpg)

ClubNews/update.php里面

```php+HTML
<!--                     这里下面是把payType表里面的缴费类型进行遍历填写，并把课程名称填入一起放到hash表中
                    并实时显示 -->
                    <?php
                    $typeArray = PayType::model()->findAll();/*从payType取数据*/
                    for($i=0;$i<count($typeArray);$i=$i+2){?><!-- 因为要一行显示两栏所以是i+2 -->
                    <tr>
                        <td><?php echo $typeArray[$i]->pay_type;?></td>
                        <td >
                            <?php echo $form->textField($model, 'price['.$i.']', array('class' => 'input-text price')); ?>
                            <?php echo $form->error($model, 'price['.$i.']', $htmlOptions = array()); ?>
                        </td>

                        <?php if($i+1<count($typeArray)){?> <!-- 此处防止出现灰色 -->
                         <td><?php echo  $typeArray[$i+1]->pay_type;?></td>
                        <td >
                            <?php echo $form->textField($model, 'price['.($i+1).']', array('class' => 'input-text price')); ?>
                            <?php echo $form->error($model, 'price['.($i+1).']', $htmlOptions = array()); ?>
                        </td>
                     <?php 
                        }else{?>
                            <td colspan="2"></td>
                        <?php }?>
```

```js
    
//这个放在<script></script>里面
		$('.price').keydown((e)=>{  // 实时监听键盘事件  实现实时计算总钱数
        //console.log($('.price'))
        let price = $('.price')
        let sum = 0
        //console.log(price.length)
        for(let i =0 ;i<price.length;i++){
            console.log(price[i])
            sum += parseInt($(price[i]).val()) //把字符串转化为数字，并取出price里的值进行累加并赋给sum
        }
        $('#ClubNews_cost').val(sum) //把sum赋给Clubnews_cost
    })
```

model/ClubNews.php

```php
class ClubNews extends BaseModel {
    public $news_content_temp = '';
    public $price=array();//定义一个price数组  用于储存该页面中每一种缴费类型的钱数
    public function tableName() {
        return '{{course}}';
    }
```

ClubNewsController.php

```php
    public function actionUpdate($id) {
        $modelName = $this->model;
        $model = $this->loadModel($id, $modelName);
        if (!Yii::app()->request->isPostRequest) {
           $data = array();
           $data['model'] = $model;
           $model->price = $this->getPrice();  /*给price数组赋值*/
           $this->render('update', $data);
        } else {
            $temp=$_POST[$modelName];
           $this-> saveData($model,$temp);
        }
    }


    public function getPrice(){
        $typeArray = PayType::model()->findAll();
        $ir = 0;
        $res = array();/*定义一个数组储存从hash表里面对应缴费类型的费用*/
        foreach($typeArray as $type){
           $res[$ir]=0; /*初始化为0，避免没找到的情况*/
           $tmp = ClubPaySet::model()->find('club_id='.$id." and pay_type='".$type->pay_type."'");
          /* 从ClubPaySet对应的数据库中找到缴费类型匹配的实例化对象*/
           if($tmp){
              $res[$ir]= $tmp->pay_num;
           }
           $ir++;
        }
        return $res;
    }



    function saveData($model,$post) {
          $s1=0;
          $model->attributes =$post;
           $model->content=$post["news_content_temp"];

           if($model->club_id){
             $temp=ClubList::model()->find("id=".$model->club_id);
              if($temp){
               $model->news_club_name=$temp->club_name;
              }
           }
           $s1=$model->save();
           $this->SavePrice($model,$post['price']);
           show_status($s1,'保存成功', get_cookie('_currentUrl_'),'保存失败');  
           
     }

     public function SavePrice($model,$price){   /*保存费用*/
        $tmp = PayType::model()->findAll();
        $i = 0;
        foreach ($tmp as $v){
            $tmp2 = ClubPaySet::model()->find('club_id='.$model->id." and pay_type='".$v->pay_type."'"
/*                从ClubPaySet的对应表中找到项目类型(club_id)对应本项目类型(id)且缴费类型对应PayType里面所有缴费类型的  一个实例化对象*/
        );
            if(!$tmp2)
                $tmp2 = new ClubPaySet();
           /* 如果找不到就new一个*/

            $tmp2->pay_num = $price[$i];
            $tmp2->pay_type = $v->pay_type;
            $tmp2->club_type = $model->name;
            $tmp2->club_id = $model->id;
            $tmp2->save();
            /*对该实例化对象进行赋值并保存*/
            $i++;
        }

     }
```

### 需求二：页面跳转到另一个页面并传id

**要求实现：从ClubNews（课程）页面设置一个按钮实现跳转到Coursedata（日程）页面，并在新页面只显示父页面课程下的日程，并在新页面实现为父课程页面添加新的日程**

![image-20211213093936601](https://s3.bmp.ovh/imgs/2021/12/bfde92b4ecb5cbee.jpg)

ClubNews/index.php

```php+HTML
    <td style='text-align: center;'>
        <a class="btn btn-blue" href="<?php echo $this->createUrl('Coursedata/index', array('id'=>$v->id,));?>" title="日程编辑"><i class="fa fa-table "> 日程查看</i></a>
        <!--这里把id传过去-->
    </td>
```

CoursedataController.php

```php
     public function actionIndex( $id="0") {
        set_cookie('_currentUrl_', Yii::app()->request->url);
        $modelName = $this->model;
        $model = $modelName::model();
        if($id!="0")  //如果id传过来了
        {
            $a = ClubNews::model()->find('id='.$id);//接收从父页面传来的id对应的那一个实例化对象
            $model->workyear = $a->f_year; 
            $model->workterm = $a->f_term;
            $model->course_name = $a->name;

        }
        $criteria = new CDbCriteria;

        $criteria->condition=get_where('1=1',$model->course_name,'course_name',$model->course_name,'"');


        //1=1 and workyear="2020-2021" and workterm="上学期" and workcourse="程序设计基础" and workteacher="曾锡山"
        //put_msg("21"." ".$criteria->condition);
        /*criteria为筛选条件，更改对条件即可完成筛选，第一个不用改，第二个改成index里面对应命名
        （即参数，应设置为默认0），第三个为此模块中的筛选的表名，第四个为index里面对应命名（即参数）*/
        parent::_list($model, $criteria, 'index', array()); //调用S
    }

```

Coursedata/index.php

```php

        <form action="<?php echo Yii::app()->request->url;?>" method="get">


     <?php $_SESSION["workyear"]=$model->workyear;
     $_SESSION["workterm"]=$model->workterm;
     /*$_SESSION["workcourse"]=$model->workcourse;*/
     /*$_SESSION["workteacher"]=$model->workteacher;*/
     $_SESSION["course_name"]=$model->course_name;?>  <!-- 储存session，供create页面使用 -->
     </form>
```

Coursedata/update.php

```php
                    <tr>
                        <td ><?php echo $form->labelEx($model, 'course_name'); ?></td>
                        <td >
                            <?php echo $form->textField($model, 'course_name', array('class'=>'input-										text','value'=>$_SESSION["course_name"],'readonly'=>true)); ?>      
                            <?php echo $form->error($model, 'course_name', $htmlOptions = array()); ?>
                        </td>
                    </tr>
                    <tr>
                        <td ><?php echo $form->labelEx($model, 'workyear'); ?></td>
                        <td >
                            <?php echo $form->textField($model, 'workyear', array('class'=>'input-											text','value'=>$_SESSION["workyear"],'readonly'=>true)); ?>      
                            <?php echo $form->error($model, 'workyear', $htmlOptions = array()); ?>
                        </td>
                    </tr>
                    <tr>
                        <td ><?php echo $form->labelEx($model, 'workterm'); ?></td>
                        <td >
                            <?php echo $form->textField($model, 'workterm', array('class'=>'input-											text','value'=>$_SESSION["workterm"],'readonly'=>true)); ?>      
                            <?php echo $form->error($model, 'workterm', $htmlOptions = array()); ?>
                        </td>
                    </tr>
```

### 需求三：页内导航栏

需求实现在一个页面内查看不同类型的数据

![image-20211215145459155](https://s3.bmp.ovh/imgs/2021/12/d8782fe890e146f8.jpg)

Comment/index.php

```php+HTML
<?php if (!isset($_REQUEST['type'])) {$_REQUEST['type']=0;} ?>
<div class="box">
     <div class="box-title c"><h1><i class="fa fa-table"></i>评价查看列表</h1></div><!--box-title end-->
    <div class="box-content">
        <div class="box-header">
            <a class="btn" href="javascript:;" onclick="we.reload();"><i class="fa fa-refresh"></i>刷新</a>
            <a style="display:none;" id="j-delete" class="btn" href="javascript:;" onclick="we.dele(we.checkval('.check-item input:checked'), deleteUrl);"><i class="fa fa-trash-o"></i>刪除</a>
        </div><!--box-header end-->

<?php if($_SESSION['courseid']){ 
    //如果$_SESSION['courseid']有值，即用户选择了一个课程那就只显示该课程的
    //count是计算该分数下有多少条
    $courseid=$_SESSION['courseid'];
    $count1=count(Comment::model()->findALL('score=1 and courseid='.$courseid));
$count1p=count(Comment::model()->findALL('score=1.5 and courseid='.$courseid));
$count2=count(Comment::model()->findALL('score=2 and courseid='.$courseid));
$count2p=count(Comment::model()->findALL('score=2.5 and courseid='.$courseid));
$count3=count(Comment::model()->findALL('score=3 and courseid='.$courseid));
$count3p=count(Comment::model()->findALL('score=3.5 and courseid='.$courseid));
$count4=count(Comment::model()->findALL('score=4 and courseid='.$courseid));
$count4p=count(Comment::model()->findALL('score=4.5 and courseid='.$courseid));
$count5=count(Comment::model()->findALL('score=5 and courseid='.$courseid));
}
else {//如果没有则显示全部
    $count1=count(Comment::model()->findALL('score=1'));
$count1p=count(Comment::model()->findALL('score=1.5'));
$count2=count(Comment::model()->findALL('score=2'));
$count2p=count(Comment::model()->findALL('score=2.5'));
$count3=count(Comment::model()->findALL('score=3'));
$count3p=count(Comment::model()->findALL('score=3.5'));
$count4=count(Comment::model()->findALL('score=4'));
$count4p=count(Comment::model()->findALL('score=4.5'));
$count5=count(Comment::model()->findALL('score=5'));
} ?>




    <div class="box-search">
        <form action="<?php echo Yii::app()->request->url;?>" method="get">
            <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">
                            
            <span>课程名称</span>
            <input style="width:200px;" class="input-text" id="course_name" type="text" readonly='ture'>
            <input type="hidden" style="width:200px;" id="course_id" class="input-text" type="text" name="courseid">

                <input id="course_select_btn" class="btn" type="button" value="选择">
                <button class="btn btn-blue" type="submit">查询</button>
            </form>

            <div class="box-detail-tab box-detail-tab mt15">
            <ul class="flexbox">
                <?php $action=$_REQUEST['type'];?>
                <li<?php if($action==1){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=1',array('courseid'=>$_SESSION['courseid']));?>">1星<?php echo '('.$count1.')'?></a>
                </li>
                <li<?php if($action==1.5){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=1.5',array('courseid'=>$_SESSION['courseid']));?>">1.5星<?php echo '('.$count1p.')'?></a>
                </li>
                <li<?php if($action==2){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=2',array('courseid'=>$_SESSION['courseid']));?>">2星<?php echo '('.$count2.')'?></a>
                </li>
                <li<?php if($action==2.5){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=2.5',array('courseid'=>$_SESSION['courseid']));?>">2.5星<?php echo '('.$count2p.')'?></a>
                </li>
                <li<?php if($action==3){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=3',array('courseid'=>$_SESSION['courseid']));?>">3星<?php echo '('.$count3.')'?></a>
                </li>
                <li<?php if($action==3.5){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=3.5',array('courseid'=>$_SESSION['courseid']));?>">3.5星<?php echo '('.$count3p.')'?></a>
                </li>
                <li<?php if($action==4){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=4',array('courseid'=>$_SESSION['courseid']));?>">4星<?php echo '('.$count4.')'?></a>
                </li>
                <li<?php if($action==4.5){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=4.5',array('courseid'=>$_SESSION['courseid']));?>">4.5星<?php echo '('.$count4p.')'?></a>
                </li>
                <li<?php if($action==5){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('Comment/index&type=5',array('courseid'=>$_SESSION['courseid']));?>">5星<?php echo '('.$count5.')'?></a>
                </li>
            </ul>
        </div><!--box-detail-tab end-->

               <!--  <td><//?php echo $form->labelEx($model, 'courseid'); ?></td>
                    <td>
                     <//?php echo $form->textField($model, 'courseid', array('class' => 'input-text-add','readonly'=>'ture')); ?>
                        <input id="club_select_btn" class="btn" type="button" value="选择">
                        <//?php echo $form->error($model, 'courseid', $htmlOptions = array()); ?>
                    </td> -->
</div><!--box-search end-->

<div class="box-table">
    <table class="list">
<thead>

    <tr>
        <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
        <th style='text-align: center;'>编号</th>
        <th style='text-align: center;'>用户id</th>
        <th style='text-align: center;'>报名课程</th>
        <th style='text-align: center;'>报名时间</th>
        <th style='text-align: center;'>评分（点击查看评价）</th>
        <th style='text-align: center;'>操作</th>
    </tr>
</thead>
        <tbody>

<?php 
$index = 1;
foreach($arclist as $v){
    if($v->status!=2) continue; //如果未评分则不予显示
?>
<tr>
    <td class="check check-item"><input class="input-check" type="checkbox" value="<?php echo CHtml::encode($v->id); ?>"></td>
    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
   <td style='text-align: center;'><?php echo $v->userid; ?></td>
   <td style='text-align: center;'><?php echo ClubNews::model()->find("id=".$v->courseid)->name; ?></td>
    <td style='text-align: center;'><?php echo $v->registrationtime; ?></td>
    <?php if($v->score) {?>
        <td style='text-align: center;'><a onclick="read_score('<?php echo $v->id;?>')" title="评分查看" href="javascript:;"><?php echo $v->score; ?>星</a></td><?php }else {?>
            <td style='text-align: center;'>未评分</td><?php }?>
    <td style='text-align: center;'>
    <a class="btn" href="javascript:;" onclick="we.dele('<?php echo $v->id;?>', deleteUrl);" title="删除"><i class="fa fa-trash-o"></i></a>
    </td>
</tr>
<?php $index++; } ?>
                </tbody>
            </table>
        </div><!--box-table end-->
        <div class="box-page c"><?php $this->page($pages);?></div>
        
    </div><!--box-content end-->
</div><!--box end-->

<script>
var deleteUrl = '<?php echo $this->createUrl('delete', array('id'=>'ID'));?>';
$('#course_select_btn').on('click', function(){ read_course(); });
    
 function read_course(){
        $.dialog.data('id', 0);
        //console.log($.dialog.data('id'));
        $.dialog.open('<?php echo $this->createUrl("select/course");?>',{
            id:'course',lock:true,opacity:0.3,width:'500px',height:'60%',
            title:'选择课程',
            close: function () {
                //console.log($.dialog.data('id'));
            if($.dialog.data('id')>0){
               $('#course_name').val($.dialog.data('name'));
               $('#course_id').val($.dialog.data('id'));
            }
        }
    });    
  }

function read_score(sid){   //如果点击了查看按钮就调用这个函数并穿了一个id
        $.dialog.data('id', 0);
        //console.log(sid);
        var url = '<?php echo $this->createUrl("SignList/score", array('id'=>'ID'));?>'
        url = url.replace(/ID/, sid); /*替换ID为选中的记录的id*/
        $.dialog.open(url,{
            id:'score',lock:true,opacity:0.3,width:'500px',height:'60%',
            title:'评分明细',
            close: function () {
                //console.log($.dialog.data('id'));
        }
    });    
  }
</script>


```

CommentController.php

```php
    public function actionIndex($courseid = '',$type='') {
    set_cookie('_currentUrl_', Yii::app()->request->url);
    //$type=$_REQUEST['type'];
    $_SESSION['courseid']=$courseid;
    $modelName = $this->model;
    $model = $modelName::model();
    $criteria = new CDbCriteria;
    $data = array();
    $criteria->condition=get_where('1=1',$courseid,'courseid',$courseid,'"');
    $criteria->condition=get_where($criteria->condition,$type,'score',$type,'"');
    parent::_list($model, $criteria, 'index', $data,20);
    }
```

SignList/score.php

```php+HTML
<div class="box">
    <div class="box-content">
        <div>
            <form action="<?php echo Yii::app()->request->url;?>" method="get">
                <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">
            </form>
        </div><!--box-search end-->
        

<div class="box-table">
    <table class="list">
    <thead>
    <tr>
        <th style='text-align: center;'>评分</th>
    </tr>
    </thead>
    <tbody>
<?php 
foreach($arclist as $v){
?>
    <tr>
        <td style='text-align: center;'><?php echo $v->score; ?></td>
    </tr>
<?php  } ?>
        </tbody>
    </table>
</div><!--box-table end-->

<div class="box-table">
    <table class="list">
        <thead>
        <tr>
            <th style='text-align: center;'>评价详情</th>
        </tr>
        </thead>
        <tbody>
    <?php
    $index = 1;
    foreach($arclist as $v){
    ?>
        <tr>
            <td style='text-align: center;'><?php echo $v->comment; ?></td>
        </tr>
                <?php $index++; } ?>
            </tbody>
        </table>
    </div>
        <div class="box-page c"><?php $this->page($pages); ?></div>
    </div><!--box-content end-->
</div><!--box end-->
<script>
$(function(){
    api = $.dialog.open.api;	// 			art.dialog.open扩展方法
    if (!api) return;

    // 操作对话框
    api.button(
        {
            name: '确认'
        }
    );
});
</script>
```

### 需求四：文件/图片上传

![](https://s3.bmp.ovh/imgs/2021/12/cceda384b0ec7522.jpg)

![](https://s3.bmp.ovh/imgs/2021/12/3567e507f1ad717b.jpg)

需求：1 实现上传图片并可以看其缩略图，如果上传文件则可以看到其对应格式的图标，点击图标可下载图片

​			2 实现如果用户上传了不符合格式的东西可以显示警告图标并禁止用户下载，增加系统的鲁棒性

studentfile/index.php

```php+HTML
 <td style='text-align: center;'><?php echo BaseLib::model()->show_pic($v->cpath);?></td>
```

studentfile/update.php

```php+HTML
                    <tr>

                         <td><?php echo $form->labelEx($model, 'cpath'); ?></td>
                        <td>
                            <?php echo $form->hiddenField($model, 'cpath', array('class' => 'input-text fl')); ?>
                            <div>只能上传 doc docx pdf zip rar 格式文件</div>
                            <!-- 改缩略图这里要改 -->
                            <!-- face_game_bigpic -->
                            <?php /*$basepath=BasePath::model()->getPath();*/
                            $picprefix='';
                            //$model->news_pic='t1234.jpg';
                            //if($basepath){ $picprefix=$basepath; }?>
                         <div class="upload_img fl" id="upload_pic_studentfile_cpath"> 
                          <?php if(!empty($model->cpath)) {?>
                             <a href="<?php  if(substr($model->cpath,-3,3)=='pdf' || substr($model->cpath,-4,4)=='docx' || substr($model->cpath,-3,3)=='doc' || substr($model->cpath,-3,3)=='zip' || substr($model->cpath,-3,3)=='rar')
                                     echo $model->cpath;
                              else
                                     echo   'https://z3.ax1x.com/2021/11/06/IMh0XT.png'; ?>" target="_blank">
                             <img src="<?php if (substr($model->cpath,-3,3)=='pdf') 
                                echo '/hsreport/uploads/image/pdf.png';
                                else if(substr($model->cpath,-4,4)=='docx')
                                echo '/hsreport/uploads/image/WORD.png';
                                else if(substr($model->cpath,-3,3)=='doc')
                                echo '/hsreport/uploads/image/WORD.png';
                                else if(substr($model->cpath,-3,3)=='zip')
                                echo '/hsreport/uploads/image/zip.png';
                                else if(substr($model->cpath,-3,3)=='rar')
                                echo '/hsreport/uploads/image/rar.png';
                                else 
                                echo '/hsreport/uploads/image/fail.png';
                                ?>", width="50">
                             </a>
                             <?php }?>
                             </div>
                            <script>we.uploadpic('<?php echo get_class($model);?>_cpath','<?php echo $picprefix;?>','','','',0);</script>
                            <?php echo $form->error($model, 'cpath', $htmlOptions = array()); ?>
                        </td>
                    </tr>

```

models/BaseLib.php

```php+HTML
function show_pic($flie='',$id=''){
    $html='';
    if(strlen($flie)>40){
        $html=empty($id)?'<div style="text-align:center">':
            '<div style="float: left; margin-right:10px" id="upload_pic_'.$id.'">';
        if(substr($flie,-3,3)=='pdf' || substr($flie,-4,4)=='docx' || substr($flie,-3,3)=='doc' 
           || substr($flie,-3,3)=='zip' || substr($flie,-3,3)=='rar')
        $html.= '<a href="'.$flie.'" target="_blank" title="点击查看">';
        else
           $html.= '<a href="https://z3.ax1x.com/2021/11/06/IMh0XT.png" target="_blank" title="格式错误">';
         //这里防止用户下载错误格式的文件，增加程序鲁棒性


        $html.= substr($flie,-3,3)=='pdf'?
            '<img src="'.'/hsreport/uploads/image/pdf.png'.'" style="max-height:30px; max-width:20px;">':'';
         $html.= substr($flie,-4,4)=='docx'?
            '<img src="'.'/hsreport/uploads/image/WORD.png'.'" style="max-height:30px; max-width:20px;">':'';
        $html.= substr($flie,-3,3)=='doc'?
            '<img src="'.'/hsreport/uploads/image/WORD.png'.'" style="max-height:30px; max-width:20px;">':'';
         $html.= substr($flie,-3,3)=='zip'?
            '<img src="'.'/hsreport/uploads/image/zip.png'.'" style="max-height:30px; max-width:20px;">':'';
         $html.= substr($flie,-3,3)=='rar'?
            '<img src="'.'/hsreport/uploads/image/rar.png'.'" style="max-height:30px; max-width:20px;">':'';
         $html.= substr($flie,-4,4)!='docx' && substr($flie,-3,3)!='pdf' && substr($flie,-3,3)!='doc' && substr($flie,-3,3)!='zip' && substr($flie,-3,3)!='rar' ?
            '<img src="'.'/hsreport/uploads/image/fail.png'.'" style="max-height:30px; max-width:20px;text-align:center;">'.'文件格式错误':'';
        $html.='</a></div>';
    }
    return $html;
}

```

