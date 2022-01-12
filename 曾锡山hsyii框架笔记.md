

## 曾锡山HsYii框架笔记

[TOC]



### 需求一：在ClubNews页面上取出PayType对应的数据库(pay_type)里的缴费类型进行遍历填写并结合ClubNews的项目类型一起传到ClubPaySet对应的数据库(hash)

![]()

**要求实现：**

**1，实施输入实时显示总钱数，并在下一次进入该页面时输入框内数字不发生变化**

**2，pay_type里面有多少个类型在ClubNews/update.php就要显示多少个输入框**

![](https://s3.bmp.ovh/imgs/2021/12/349bdc52fa522b6a.jpg)

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

### 需求五：一二三级菜单

![](https://s4.ax1x.com/2021/12/22/Tlq6sS.jpg)

menu_main表

![](https://s4.ax1x.com/2021/12/22/TlLuy8.jpg)

menu表

![](https://s4.ax1x.com/2021/12/22/TljKOK.jpg)

### 需求六：同一个文件实现状态不同的页面

![](https://s4.ax1x.com/2021/12/22/TlzrkD.jpg)

![](https://s4.ax1x.com/2021/12/22/T1iN1e.jpg)

实现：

在这个数据库course中加上status 的字段判断应该显示那个页面

status == 1的那个页面对应的url是ClubNews/index&news_type=1

model/ClubNews/update.php

```php+HTML
</div>
        <div class="box-detail">
     <table class="mt15">
    <?php 
     if (!isset($_REQUEST['news_type'])) {$_REQUEST['news_type']=1;} 
     if($_REQUEST['news_type']==2){   ?>
        <div class="mt15">
            <table class="table-title"><tr> <td>审核信息</td></tr></table>
            <table>
                <tr>
                    <td width="15%"><?php echo $form->labelEx($model, 'status'); ?></td>
                    <td width="35%">
                        <?php echo $form->radioButtonList($model, 'status', [4=>'通过',3=>'驳回'], array('separator'=>'', 'template'=>'<span class="radio">{input} {label}</span> ')); ?>
                        <?php echo $form->error($model, 'status'); ?>
                    </td>
                    <td width="15%"><?php echo $form->labelEx($model, 'reasons_for_failure'); ?></td>
                    <td width="35%">
                        <?php echo $form->textArea($model, 'reasons_for_failure', array('class' => 'input-text')); ?>
                        <?php echo $form->error($model, 'reasons_for_failure', $htmlOptions = array()); ?>
                    </td>
                </tr>
            </table>
        </div>
        <div class="box-detail-submit">
          <button onclick="submitType='queren'" class="btn btn-blue" type="submit">确认</button>
          <button class="btn" type="button" onclick="we.back();">取消</button>
         </div>
      <?php }
      else if ($_REQUEST['news_type']==1 || $_REQUEST['news_type']==3){
      if($_REQUEST['news_type']==3){ ?>
        <table class="table-title"><tr> <td>审核状态</td></tr></table>
            <table>
                <tr>
                    <td width="15%"><?php echo $form->labelEx($model, 'status'); ?></td>
                    <td width="35%">
                        <?php echo "驳回" ?>
                    </td>
                    <td width="15%"><?php echo $form->labelEx($model, 'reasons_for_failure'); ?></td>
                    <td width="35%">
                        <?php echo $model->reasons_for_failure; ?>
                        <?php echo $form->error($model, 'reasons_for_failure', $htmlOptions = array()); ?>
                    </td>
                </tr>
            </table>
        <?php }?>
        <div class="box-detail-submit">
          <button onclick="submitType='baocun'" class="btn btn-blue" type="submit">保存</button>
          <button onclick="submitType='shenhe'" class="btn btn-blue" type="submit">提交审核</button>
          <?php if($_REQUEST['news_type']!=1) { ?>
            /*只有提交了才能撤销提交*/
          <button onclick="submitType='chexiao'" class="btn btn-blue" type="submit">撤销提交</button>
          <?php } ?>
          <button class="btn" type="button" onclick="we.back();">取消</button>
         </div>
     <?php }
     else {?>
        <div class="box-detail-submit">
          <button onclick="submitType='queren'" class="btn btn-blue" type="submit">确认</button>
          <button onclick="submitType='chexiao'" class="btn btn-blue" type="submit">撤销提交</button>
          <button class="btn" type="button" onclick="we.back();">取消</button>
         </div>
     <?php }?>
 </table>
        </div>
    </div>
</div><!--box-detail-bd end-->
```

ClubNews/index.php

```php+HTML
<?php if (!isset($_REQUEST['news_type'])) {$_REQUEST['news_type']=0;} ?>
<?php
 $years=base_year::model()->findALL();
 $terms=base_term::model()->findALL();
?>

..........

<div class="box-table">
    <table class="list">
<thead>

    <tr>
        <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
        <th style='text-align: center;'>编号</th>
        <th style='text-align: center;'>研学课程名称</th>
        <th style='text-align: center;'>类型</th>
        <th style='text-align: center;'>简要介绍</th>
        <th style='text-align: center;'>报名开始时间</th>
        <th style='text-align: center;'>报名结束时间</th>
        <th style='text-align: center;'>研学课程开始时间</th>
        <th style='text-align: center;'>研学课程结束时间</th>
        <th style='text-align: center;'>缩略图</th>
        <th style='text-align: center;'>状态</th>
        <th style='text-align: center;'>缴费总额</th>
        <th style='text-align: center;'>日程设置</th>
        <th style='text-align: center;'>操作</th>
    </tr>
</thead>
<tbody>
<?php 
$index = 1;
foreach($arclist as $v){ ?>
<tr>
    <td class="check check-item"><input class="input-check" type="checkbox" value="<?php echo CHtml::encode($v->id); ?>"></td>
    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
    <td style='text-align: center;'><?php echo $v->name; ?></td>
    <td style='text-align: center;'><?php echo $v->type; ?></td>
    <td style='text-align: center;'><?php echo $v->introduce; ?></td>
    <td style='text-align: center;'><?php echo $v->sign_date_start; ?></td>
    <td style='text-align: center;'><?php echo $v->sign_date_end; ?></td>
    <td style='text-align: center;'><?php echo $v->signIn_date_start; ?></td>
    <td style='text-align: center;'><?php echo $v->signIn_date_end; ?></td>
    <td style='text-align: center;'><?php echo BaseLib::model()->show_pic($v->imagesurl);?></td>
    <td style='text-align: center;'><?php switch($v->status){
                                                                case 1:echo "保存"; break;
                                                                case 2:echo "提交审核";break;
                                                                case 3:echo "驳回";break;
                                                                case 4:echo "审核通过";break;
                                                                case 11:echo "报名进行中";break;
                                                                case 12:echo "报名已结束";break;
                                                                case 21:echo "活动进行中";break;
                                                                case 22:echo "活动已结束";break;
                                                                default:
                                                                    echo "";
                                                            }
     ?></td>
     <td style='text-align: center;'><?php echo $v->cost ?></td>
    <td style='text-align: center;'>
        <a class="btn btn-blue" href="<?php echo $this->createUrl('Coursedata/index', array('id'=>$v->id,));?>" title="日程编辑"><i class="fa fa-table "> 日程查看</i></a>
    </td>
    <td style='text-align: center;'>
     
        <a class="btn" href="<?php echo $this->createUrl('update', array('id'=>$v->id,'news_type'=>Yii::app()->request->getParam('news_type')));?>" title="编辑"><i class="fa fa-edit"></i></a>
        /*上面记得把news_type传给update */
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
```

model/ClubNews.php

```php
        /*status:状态（1保存 2提交 3驳回 4通过   11报名开始 12报名结束  21活动开始 22活动结束）*/
    protected function afterFind() {
        parent::afterFind();
        $this->news_content_temp = $this->content;
        date_default_timezone_set("PRC");
        $today=date("Y-m-d H:i:s");
        if($this->status>=4)  /*当通过审核时*/
        {
            if($today<$this->sign_date_start) ;  /* 未开始报名的活动*/
            else if($today<=$this->sign_date_end) $this->status=11; /*报名中的活动*/
            else if($today<=$this->signIn_date_start) $this->status=12;/*未开始进行的活动*/
            else if($today<=$this->signIn_date_end) $this->status=21;/*正在进行的活动*/
            else $this->status=22; /*已结束的活动*/
        }
        
        $this->save();
        return true;
    } 

    protected function beforeSave() {
        parent::beforeSave();
        if(isset($_POST['submitType']))
        {
           $status=$_POST['submitType'];
         if($status=='shenhe')
         {
            $this->status=2; 
         }
         else if($status=='queren')
         {
            
         }
         else if($status == 'chexiao')
         {
             $this->status=1; //撤销的活动
         }
         else
         {
            $this->status=1;
         } 
        }
         
        // 圖文描述處理
        return true;
    }

```

ClubNewsController.php

```php
/*status:状态（1保存 2提交 3驳回 4通过   11报名开始 12报名结束  21活动开始 22活动结束）*/
    public function actionIndex($styear="",$sterm="",$cstart_date="",$cend_date="",$start_date="",$end_date="",$news_type="") {
        //这里一定要传入一个news_type
    set_cookie('_currentUrl_', Yii::app()->request->url);
    $modelName = $this->model;
    $model = $modelName::model();
    $criteria = new CDbCriteria;
    $w1=get_where('1=1',$styear,'f_year',$styear,'"');
    $criteria->condition=get_where($w1,$sterm,'f_term',$sterm,'"');
    $criteria->condition=get_where($criteria->condition,$cstart_date,'starttime>=',$cstart_date,'"');
    $criteria->condition=get_where($criteria->condition,$cend_date,'starttime<=',$cend_date,'"');
    $criteria->condition=get_where($criteria->condition,$start_date,'registrationstartdate>=',$start_date,'"');
    $criteria->condition=get_where($criteria->condition,$end_date,'registrationstartdate<=',$end_date,'"');
    $criteria->condition=get_where($criteria->condition,$news_type,'status',$news_type,'"'); 
     //put_msg($criteria->condition);
    $data = array();
    parent::_list($model, $criteria, 'index', $data,20);
    }
```

### 需求七：导入excel文件并把数据显示出来

![](https://s4.ax1x.com/2021/12/24/TNryjJ.jpg)

![](https://s4.ax1x.com/2021/12/24/TNrWAx.jpg)

![](https://s4.ax1x.com/2021/12/24/TNrqHI.jpg)

Userinfo/import.php

```php
<div class="box">
    <div class="box-title c">
    <h1><i class="fa fa-table"></i>学生信息》导入页面</h1><span class="back">
    <a class="btn" href="javascript:;" onclick="we.back();">
    <i class="fa fa-reply"></i>返回</a></span></div><!--box-title end-->
    <div class="box-detail">
     <?php  $form = $this->beginWidget('CActiveForm', get_form_list()); ?>
        <div class="box-detail-bd">
            <div style="display:block;" class="box-detail-tab-item">
                <table class="mt15">
                    <tr>
                        <td><?php echo "导入模板下载"; ?></td>
                        <td>
                            <?php $mobanUrl = '/sanli/uploads/moban.xlsx'?>
                            <a href="<?php echo $mobanUrl ?>">模板下载</a>
                        </td>
                    </tr>
                     <tr>
                         <td><?php echo "文件上传"; ?></td>
                        <td>
                            <?php echo $form->hiddenField($model, 'excelPath', array('class' => 'input-text fl')); ?>   
                            <div>只能上传 xlsx xls 格式文件</div>
                            <!-- 改缩略图这里要改 -->
                            <!-- face_game_bigpic -->
                            <?php /*$basepath=BasePath::model()->getPath();*/
                            $picprefix='';
                            //$model->news_pic='t1234.jpg';
                            //if($basepath){ $picprefix=$basepath; }?>
                         <div class="upload_img fl" id="upload_pic_cstuifo_excelPath"> </div>

                            <script>we.uploadpic('<?php echo get_class($model);?>_excelPath','<?php echo $picprefix;?>','','','',0);</script>
                            <?php echo $form->error($model, 'excelPath', $htmlOptions = array()); ?>
                        </td>
                    </tr>

                </table>
        </div>
        <div class="box-detail-submit">
          <button onclick="submitType='baocun'" class="btn btn-blue" type="submit">导入</button>
          <button class="btn" type="button" onclick="we.back();">取消</button>
         </div>
         
    
            <?php $this->endWidget();?>
  </div><!--box-detail end-->
</div><!--box end-->
 
   
```

UserinfoController.php

```php
     function saveData($model,$post,$is_import=0) {
        $model->attributes = $post;
        if($is_import){
            $path=ROOT_PATH.'/uploads/temp/'. $post['excelPath'];
            $nameArray=array(
                'name','gender','education',
                'nikename','phone','schoolname',
                'grade','class',
                 'idnum','parents','p_phone',);
            //从B列 3行开始导入
            $Import = new ImportExcel();
            $status=$Import->import($path,$this->model,$nameArray,'B',3);
        }
        show_status($status,'保存成功', get_cookie("_currentUrl_"),'文件类型错误');
    }
```

models/Userinfo.php

```php
  public $excelPath="";
```

models/importExcel.php

```php+HTML
<?php


class ImportExcel
{

    function isEndOf($str,$hzArray=array()){
        foreach ($hzArray as $hz){
            if ($this->file_hz($str)==$hz)return true; //判断后缀名是不是xlsx 或xls
        }
        return false;
    }

    function file_hz($file){  //求出文件后缀名的函数
        //put_msg('hzhzhz');
        //put_msg(substr($file, strrpos($file, '.')+1));
        return substr($file, strrpos($file, '.')+1);
    }

    public function import($excelFile = "",$modelName='',$nameArray=array(),$colFirst='B',$rowFirst=3)
    {
        if(!$this->isEndOf($excelFile,array('xls','xlsx'))) //拦截文件格式不正确的
            return false;

        Yii::$enableIncludePath = false;
        Yii::import('application.extensions.PHPExcel.PHPExcel', 1);
        $extension=$this->file_hz($excelFile);
        if ($extension=='xls') {
            $excelReader = PHPExcel_IOFactory::createReader('Excel5');
        } else {
            $excelReader = PHPExcel_IOFactory::createReader('Excel2007');
        }

        $objPHPExcel = $excelReader->load($excelFile);
        $sheet = $objPHPExcel->getSheet(0);
        $highestRow = $sheet->getHighestRow(); // 取得总行数
        //nameArray是传进来需要导入赋值的变量名数组
        for ($row = $rowFirst; $row <= $highestRow; $row++){
            $a = new $modelName();
            for($i=0 ; $i<count($nameArray);$i++)
            {
                //colFirst是起始列数
                $col = chr(ord($colFirst)+$i);
                //ord是把字符转换为ASCII chr是转换为字符
                $a->{$nameArray[$i]}=$sheet->getCell($col.$row)->getValue();
                //加大括号是把字符串变成变量名
            }
            $a->save();
        }
        return true;
    }
}
```

### 需求八：实现人员分组功能

![](https://s4.ax1x.com/2021/12/30/T2UgNF.jpg)

在这个页面点击分组，实现自动把报名的学生（registration表）进行分组分房分车，点击重置分组实现清空分组

分组前 courseid为课程编号  groupnum为组号  room_id为房间内部编号  car_id为车内部编号

![](https://s4.ax1x.com/2021/12/30/T2apHf.jpg)

分组规则：

同一个班的优先分到一个组里，一个房间里尽可能是一个组的而且性别必须一样， 同一辆车里的尽可能是一个组的，剩余的人员单独分。用户输入每组人数，每车人数，每个房间人数。

分组后：

![](https://s4.ax1x.com/2021/12/30/T2dyYF.jpg)

**实现算法：**



建立group car room三个基本表

![](https://s4.ax1x.com/2021/12/30/T2dJJg.jpg)

![](https://s4.ax1x.com/2021/12/30/T2doFO.jpg)

![](https://s4.ax1x.com/2021/12/30/T2dqld.jpg)

SignList/group.php

```php
<div class="box">
        
    <div class="box-detail">
        <?php  $form = $this->beginWidget('CActiveForm', get_form_list()); ?>
        <div class="box-detail-bd">
            <div style="display:block;" class="box-detail-tab-item">

    <form action="<?php echo Yii::app()->request->url;?>" method="get">
    <?php
     $_SESSION["coursename"]=$model->coursename;
     $_SESSION["courseid"]=$model->courseid;
     ?>  <!-- 储存session，供create页面使用 -->
     </form>
                <table class="mt15">
                    <tr>
                        <td ><?php echo "分组人数  ".$model->coursename." ".$model->courseid; ?></td>
                        <td>
                            <?php echo $form->textField($model, 'groupnum',array('class' => 'input-text')); ?>
                            <?php echo $form->error($model, 'groupnum', $htmlOptions = array()); ?>
                        </td>
                    </tr>
                </table>
                <div class="box-detail-submit">
                <!-- <button onclick="submitType='group'" class="btn btn-blue" type="submit">分组</button> -->
                <a class="btn btn-blue" href="<?php echo $this->createUrl('DGroup');?>">分组</a>
                </div>
                <div class="box-detail-submit">
                <!-- <button onclick="submitType='group'" class="btn btn-blue" type="submit">分组</button> -->
                <a class="btn btn-blue" href="<?php echo $this->createUrl('DeleteGroup');?>">重置分组</a>
                </div>
            </div>    
        </div><!--box-detail-bd end-->
         <?php $this->endWidget();?>
    </div>
</div>
```

SignListController.php

```php
     function actionDGroup() { //分组时，男女混合，两两取值
        //$criteria =array();
        //array_push($criteria,$this->createcriteria(1),$this->createcriteria(2));    //下标0是男筛选，1是女筛选
        //$criteria ->params = array (':status' => $你的变量) ;  
        $group=Group::model()->findALL('courseid = 1'); //查找指定课程的设置的小组（可传递参数）
        foreach($group as $v)
        {
            $p_num=$v->p_num;
            $criteria=$this->createGroupcriteria($p_num);
            $result = SignList::model()->findAll($criteria);
            foreach($result as $j)
            {
                $j->groupnum=$v->g_id;
                $j->teacher=$v->teacher;
                $j->save();
            }
        }
        $this->actionDroom();
     }


     function actionDroom() { //分组时，男女混合，两两取值
        //$criteria =array();
        //array_push($criteria,$this->createcriteria(1),$this->createcriteria(2));    //下标0是男筛选，1是女筛选
        //$criteria ->params = array (':status' => $你的变量) ;  
        $room=Room::model()->findALL('courseid = 1'); //查找指定课程的设置的小组（可传递参数）
        foreach($room as $v)
        {
            $p_num=$v->people;
            $criteria=$this->createRoomcriteria($p_num);
            $result = SignList::model()->findAll($criteria);
            foreach($result as $j)
            {
                $j->room_id=$v->room_id;
                $j->save();
            }
        }
        $this->actionDCar();
     }


     function actionDCar() { //分组时，男女混合，两两取值
        //$criteria =array();
        //array_push($criteria,$this->createcriteria(1),$this->createcriteria(2));    //下标0是男筛选，1是女筛选
        //$criteria ->params = array (':status' => $你的变量) ;  
        $car=Car::model()->findALL('courseid = 1'); //查找指定课程的设置的小组（可传递参数）
        foreach($car as $v)
        {
            $p_num=$v->car_people;
            $criteria=$this->createCarcriteria($p_num);
            $result = SignList::model()->findAll($criteria);
            foreach($result as $j)
            {
                $j->car_id=$v->car_id;
                $j->save();
            }
        }
     }


     function actionDeleteGroup() { 
         
        $group=Group::model()->findALL('courseid = 1'); //只清空这个课程的分组
        foreach($group as $v)
        {
            $p_num=$v->p_num;
            $criteria=$this->createDeletecriteria($p_num);
            $result = SignList::model()->findAll($criteria);
            foreach($result as $j)
            {
                $j->groupnum=NULL;
                $j->room='';
                $j->room_id=NULL;
                $j->car='';
                $j->car_id=NULL;
                $j->teacher='';
                $j->save();
            }
        }
     }


     function createDeletecriteria($limit)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is not NULL');    //筛选还没被分组的名单，即字段group=0
        $criteria -> condition = get_where($criteria -> condition,1,'courseid',1);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        //$criteria -> order = 'grade asc,class asc';   //指定排序
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }

     function createGroupcriteria($limit)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is NULL');    //筛选还没被分组的名单，即字段group=0
        $criteria -> condition = get_where($criteria -> condition,1,'courseid',1);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        $criteria -> order = 'grade asc,class asc';   //指定排序   
        //意思是优先年级  其次优先班级进行 升序排列 
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }

     function createRoomcriteria($limit)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is not NULL and room_id is NULL');    //筛选还被分组的名单，即字段group不为空
        $criteria -> condition = get_where($criteria -> condition,1,'courseid',1);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        $criteria -> order = 'sex asc,groupnum asc';   //指定排序
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }

     function createCarcriteria($limit)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is not NULL and room_id is not NULL and car_id is NULL');    //筛选还被分组的名单，即字段group不为空
        $criteria -> condition = get_where($criteria -> condition,1,'courseid',1);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        $criteria -> order = 'groupnum asc';   //指定排序
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }
```

### 需求九：实现人员分组并查看分组结果

对于某一个课程

1.用户选择是否尽量男女混合或男女分别分组，选择是否分房间（用户指定每间房人数，每间房必须同性，而且每间房尽量同一组），选择是否分车（用户指定每辆车的载人数，每辆车尽量同一组）。而且只有已经开始的活动才能分组

2.用户可编辑组的带队老师，房间号，车牌号等信息，并自动分好。

3.用户可查看分组结果并可导出为excel。

4.每辆车和房间的内部编号自动递增。

SignList model 的registration

![](https://s4.ax1x.com/2022/01/08/7iVmVK.jpg)

![](https://s4.ax1x.com/2022/01/08/7iV1xA.jpg)

![](https://s4.ax1x.com/2022/01/08/7iV8KI.jpg)

![](https://s4.ax1x.com/2022/01/08/7iVNa8.jpg)

views/SignList文件夹下包含：group.php , grouplist.php , index.php , result.php 



**选择课程页面** （SignList/grouplist.php） 属于ClubNews的model

```php+HTML
<div class="box">
     <?php echo show_title($this,"选择课程",0);?>
    <div class="box-content">
        <div class="box-header">
            <a class="btn" href="javascript:;" onclick="we.reload();"><i class="fa fa-refresh"></i>刷新</a>
        </div><!--box-header end-->

     <form action="<?php echo Yii::app()->request->url;?>" method="get">
    <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">


<div class="box-table">
    <table class="list">
<thead>

    <tr>
        <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
        <th style='text-align: center;'>编号</th>
        <th style='text-align: center;'>研学课程名称</th>
        <th style='text-align: center;'>状态</th>
        <th style='text-align: center;'>功能</th>
    </tr>
</thead>
<tbody>
<?php 
$index = 1;
foreach($arclist as $v){ ?>
<tr>
    <td class="check check-item"><input class="input-check" type="checkbox" value="<?php echo CHtml::encode($v->id); ?>"></td>
    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
    <td style='text-align: center;'><?php echo $v->name; ?></td>
    <td style='text-align: center;'><?php switch($v->status){
                                                                case 1:echo "保存"; break;
                                                                case 2:echo "提交审核";break;
                                                                case 3:echo "驳回";break;
                                                                case 4:echo "审核通过";break;
                                                                case 11:echo "报名进行中";break;
                                                                case 12:echo "报名已结束";break;
                                                                case 21:echo "活动进行中";break;
                                                                case 22:echo "活动已结束";break;
                                                                default:
                                                                    echo "";
                                                            }
     ?></td>   
    <td style='text-align: center;'>
        <a class="btn btn-blue" href="<?php echo $this->createUrl('SignList/group', array('id'=>$v->id));?>" title="分组设置"><i class="fa fa-table "> 分组设置</i></a>
        <a class="btn btn-blue" href="<?php echo $this->createUrl('SignList/result&news_type=1', array('id'=>$v->id,));?>" title="分组结果查看"><i class="fa fa fa-check-circle"> 分组结果查看</i></a>
    </td>
</tr>
<?php $index++; } ?>
                </tbody>
            </table>
        </div><!--box-table end-->
        <div class="box-page c"><?php $this->page($pages);?></div>
    </div><!--box-content end-->
</div><!--box end-->

```

SignListController.php  

```php
    public function actionGrouplist() {
    set_cookie('_currentUrl_', Yii::app()->request->url);
    $model = ClubNews::model();  //属于ClubNews的model
    $criteria = new CDbCriteria;
    $data = array();
    $criteria->condition=get_where('1=1',4,'status>=',4,'"');
    parent::_list($model, $criteria, 'grouplist', $data,20);
    }
```

**分组设置页面**   (SignList/group.php)   属于ClubNews的model

#### 子功能一：实现选择参数并跨页面赋值

在此页面中给isMax , isCar ,isRoom赋值并传到course的表里给对应字段赋值

```php+HTML
<div class="box">
    <?php echo show_title($this,"分组设置",0)?>    
    <div class="box-detail">
        <?php  $form = $this->beginWidget('CActiveForm', get_form_list()); ?>
        <div class="box-detail-bd">
            <div style="display:block;" class="box-detail-tab-item">
            <a class="btn" href="<?php echo $this->createUrl('SignList/grouplist');?>"><i class="fa fa-reply"></i>返回</a>

     <h1 align="center">分组课程：<?php echo $model->name ?></h1>
    <h2 align="center">分组设置</h2>
    <form id="group" action="<?php echo Yii::app()->request->url;?>" method="POST">
    <div class="box-table">
            <table class="list">
                <thead>
                <tr>
                    <th style='text-align: center;'>分组项目</th>
                    <th style='text-align: center;'>是否进行该项目分组</th>
                    <th style='text-align: center;'>操作</th>
                </tr>
                </thead>
                <tbody>
                    <tr>
                        <td style='text-align: center;'>分小组</td>
                        <td style='text-align: center;'>是</td>
                        <td style='text-align: center;'><a class="btn" href="<?php echo $this->createUrl('Group/index', array('id'=>$model->id));?>" title="设置"><i class="fa fa-edit">设置</i></a></td>
                    </tr>
                    <tr>
                        <td style='text-align: center;'>是否男女混合分组</td>
                        <td style='text-align: center;'><?php echo $form->radioButtonList($model, 'isMix', [1=>'是',0=>'否'], array('separator'=>'   ', 'template'=>'<span class="radio">{input} {label}</span> ')); ?></td>
                        <td></td>
                    </tr>
                    
                    <tr>
                        <td style='text-align: center;'>分车</td>
                        <td style='text-align: center;'><?php echo $form->radioButtonList($model, 'isCar', [1=>'是',0=>'否'], array('separator'=>'   ', 'template'=>'<span class="radio">{input} {label}</span> ')); ?></td>
                        
                        <?php if($model->isCar){?>
                        <td style="display:none; text-align: center;" id="td1"></td>
                        <td style="text-align: center;" id="td2" ><a class="btn" href="<?php echo $this->createUrl('Car/index', array('id'=>$model->id));?>" title="设置"><i class="fa fa-edit">设置</i></a></td>
                        <?php }else{?>
                        <td id="td1"></td>
                        <td style="display:none; text-align: center;" id="td2" ><a class="btn" href="<?php echo $this->createUrl('Car/index', array('id'=>$model->id));?>" title="设置"><i class="fa fa-edit">设置</i></a></td>
                        <?php }?>
                    </tr>

                    <tr>
                        <td style='text-align: center;'>分房间</td>
                        <td style='text-align: center;'><?php echo $form->radioButtonList($model, 'isRoom', [1=>'是',0=>'否'], array('separator'=>'   ', 'template'=>'<span class="radio">{input} {label}</span> ')); ?></td>
                        <?php if($model->isRoom){?>
                        <td style="display:none; text-align: center;" id="td3"></td>
                        <td style=" text-align: center;" id="td4"><a class="btn" href="<?php echo $this->createUrl('Room/index', array('id'=>$model->id));?>" title="设置"><i class="fa fa-edit">设置</i></a></td>
                        <?php }else{?>
                        <td id="td3"></td>
                        <td style="display:none; text-align: center;" id="td4"><a class="btn" href="<?php echo $this->createUrl('Room/index', array('id'=>$model->id));?>" title="设置"><i class="fa fa-edit">设置</i></a></td>
                        <?php }?>
                    </tr>

                </tbody>
            </table>
            <table>
                <b>请先完成分组的所有设置，再点击下面按钮进行分组，点击后请等待弹窗提示分组完成</b>
            <div class="box-detail-submit">

                <!-- <button onclick="submitType='group'" class="btn btn-blue" type="submit">分组</button> -->
                <a class="btn btn-blue" href="<?php echo $this->createUrl('DGroup',array('isRoom'=>$model->isRoom,'isCar'=>$model->isCar,'courseid'=>$model->id,'isMix'=>$model->isMix));?>">分组</a>
                </div>
            </table>
        </div><!--box-table end-->
         <?php $this->endWidget();?>
     </form>
    </div>
</div>

<script>

$('#ClubNews_isMix_0').on('click',function(){
        $('#td1').hide();
        $('#td2').show();
        var sid=<?php echo $model->id?>;
        var url1 = "<?php echo $this->createUrl("ClubNews/update",array('id'=>'ID'))?>";
        url1 = url1.replace(/ID/, sid);
        $.ajax({
            //几个参数需要注意一下
                type: "POST",//方法类型
                dataType: "json",//预期服务器返回的数据类型
                url: url1 ,//url
                data: $('#ClubNews_isMix_0').serialize(),

            });
        
    })

    $('#ClubNews_isMix_1').on('click',function(){
        var sid=<?php echo $model->id?>;
        var url1 = "<?php echo $this->createUrl("ClubNews/update",array('id'=>'ID'))?>";
        url1 = url1.replace(/ID/, sid);
        $.ajax({
            //几个参数需要注意一下
                type: "POST",//方法类型
                dataType: "json",//预期服务器返回的数据类型
                url: url1 ,//url
                data: $('#ClubNews_isMix_1').serialize(),

            });
        
    })

    $('#ClubNews_isCar_0').on('click',function(){
        $('#td1').hide();
        $('#td2').show();
        var sid=<?php echo $model->id?>;
        var url1 = "<?php echo $this->createUrl("ClubNews/update",array('id'=>'ID'))?>";
        url1 = url1.replace(/ID/, sid);
        $.ajax({
            //几个参数需要注意一下
                type: "POST",//方法类型
                dataType: "json",//预期服务器返回的数据类型
                url: url1 ,//url
                data: $('#ClubNews_isCar_0').serialize(),

            });
    })


    $('#ClubNews_isCar_1').on('click',function(){
        $('#td1').show();
        $('#td2').hide();
        var sid=<?php echo $model->id?>;
        var url1 = "<?php echo $this->createUrl("ClubNews/update",array('id'=>'ID'))?>";
        url1 = url1.replace(/ID/, sid);
        $.ajax({
            //几个参数需要注意一下
                type: "POST",//方法类型
                dataType: "json",//预期服务器返回的数据类型
                url: url1 ,//url
                data: $('#ClubNews_isCar_1').serialize(),

            });
        
    })

    $('#ClubNews_isRoom_0').on('click',function(){
        $('#td3').hide();
        $('#td4').show();
        var sid=<?php echo $model->id?>;
        var url1 = "<?php echo $this->createUrl("ClubNews/update",array('id'=>'ID'))?>";
        url1 = url1.replace(/ID/, sid);
        $.ajax({
            //几个参数需要注意一下
                type: "POST",//方法类型
                dataType: "json",//预期服务器返回的数据类型
                url: url1 ,//url
                data: $('#ClubNews_isRoom_0').serialize(),

            });
        
    })

    $('#ClubNews_isRoom_1').on('click',function(){
        $('#td3').show();
        $('#td4').hide();
        var sid=<?php echo $model->id?>;
        var url1 = "<?php echo $this->createUrl("ClubNews/update",array('id'=>'ID'))?>";
        url1 = url1.replace(/ID/, sid);
        $.ajax({
            //几个参数需要注意一下
                type: "POST",//方法类型
                dataType: "json",//预期服务器返回的数据类型
                url: url1 ,//url
                data: $('#ClubNews_isRoom_1').serialize(),

            });
        
    })

</script>
```



```php
//SignListController.php
public function actionGroup($id=1) {
        $model = $this->loadModel($id, "ClubNews");
        if (!Yii::app()->request->isPostRequest) { //如果有表单上传
           $data = array();
           $data['model'] = $model;
           $this->render('group', $data); //重定向到group页面
        } else {
            $temp=$_POST["ClubNews"];
           $this-> saveData($model,$temp);
        }
    }
```

ClubNewsController.php  （接收传过来的值）

```php
    public function actionUpdate($id) {
        $modelName = $this->model;
        $model = $this->loadModel($id, $modelName);
        if (!Yii::app()->request->isPostRequest) {
           $data = array();
           $data['model'] = $model;
           $model->price = $this->getPrice($id);
           $this->render('update', $data);
        }
        else if(isset($_POST["ClubNews"]["isCar"])) {$model->isCar=$_POST["ClubNews"]["isCar"]; $model->save();} 
        else if(isset($_POST["ClubNews"]["isRoom"])) {$model->isRoom=$_POST["ClubNews"]["isRoom"]; $model->save();}
        else if(isset($_POST["ClubNews"]["isMix"]))  {$model->isMix=$_POST["ClubNews"]["isMix"]; 
        $model->save();}
        else {
            $temp=$_POST[$modelName];
           $this-> saveData($model,$temp);
        }
    }
```

#### 子功能二：实现在index页面实时编辑

![](https://s4.ax1x.com/2022/01/08/7iuVA0.jpg)

#### 子功能三：实现内部编号自动递增功能

由于房间内部编号指某个课程内的第几间房，内部唯一识别房间的编号，故必须在添加房间时自动生成，不能由用户输入

实现方法：在Room/index页面findAll 找到所有当前属于该课程房间的对象数组，然后计算出个数存在roomcnt

并赋值给超全局变量$_SESSION["roomcnt"]   。最后在update页面把 $_SESSION["roomcnt"]  + 1赋值给 room_id储存

**分小组设置页面**  (Room/index )    Room 的model

```c++
<?php
 $course=SignList::model()->findALL();
  //这里下面是在Room中找到该课程中目前的组数 
  $tmp=Room::model()->findAll('courseid ='.$model->courseid);
  $roomcnt = count($tmp);
?>
<div class="box">
     <?php echo show_title($this,'分房设置',0);?>
    <div class="box-content">
        <div class="box-header">
            <a class="btn" href="<?php echo $this->createUrl('create');?>"><i class="fa fa-plus"></i>添加</a>
            <a class="btn" href="javascript:;" onclick="we.reload();"><i class="fa fa-refresh"></i>刷新</a>
            <a style="display:none;" id="j-delete" class="btn" href="javascript:;" onclick="we.dele(we.checkval('.check-item input:checked'), deleteUrl);"><i class="fa fa-trash-o"></i>刪除</a>
            <a class="btn" href="<?php echo $this->createUrl('SignList/group',array("id"=>$id));?>"><i class="fa fa-reply"></i>返回</a>
        </div><!--box-header end-->

        <form action="<?php echo Yii::app()->request->url;?>" method="get">


     <?php 
     $_SESSION["course"]=$model->course;
     $_SESSION["courseid"]=$model->courseid;
      $_SESSION["roomcnt"]=$roomcnt;?>  <!-- 储存session，供create页面使用 -->
     </form>

</div><!--box-search end-->

<h1 align="center">研学课程：<?php echo $model->course ?></h1>
<h2 align="center">房间设置(房间号，酒店名称，房间人数均可双击修改)</h2>

<div class="box-table">
    <table class="list">
<thead>

    <tr>
        <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>    
        <th style='text-align: center;'>课程名称</th>
        <th style='text-align: center;'>酒店名称</th>
        <th style='text-align: center;'>房间号</th>
        <th style='text-align: center;'>内部编号(该课程的第几间房)</th>
        <th style='text-align: center;'>房间人数</th>
        <th style='text-align: center;'>操作</th>
    </tr>
</thead>
        <tbody>

<?php 
$index = 1;
foreach($arclist as $v){ 
?>
<tr>
    <td class="check check-item"><input class="input-check" type="checkbox" value="<?php echo CHtml::encode($v->id); ?>"></td>

   <td style='text-align: center;'><?php echo $v->course; ?></td>

   <?php echo"<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='hotel' thid=".$v->id.">".$v->hotel."</td>" ?>  <!--该行为实时编辑代码-->

   <?php echo"<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='room' thid=".$v->id.">".$v->room."</td>" ?>

   <td style='text-align: center;'><?php echo $v->room_id; ?></td>

   <?php echo"<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='people' thid=".$v->id.">".$v->people."</td>" ?>
    <td style='text-align: center;'>
     
        <a class="btn" href="<?php echo $this->createUrl('update', array('id'=>$v->id,));?>" title="编辑"><i class="fa fa-edit"></i></a>
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

//以下为实时编辑代码
function ShowElement(element) {
        var oldhtml = element.innerHTML;
        //创建新的input元素
        var newobj = document.createElement('input');
        //为新增元素添加类型
        newobj.type = 'text';
        //为新增元素添加value值
        newobj.value = oldhtml;
        //为新增元素添加光标离开事件
        newobj.onblur = function() {
            //当触发时判断新增元素值是否为空，为空则不修改，并返回原有值 
            element.innerHTML = this.value == oldhtml ? oldhtml : this.value;
            var sid= element.getAttribute("thid");
            var url1 = "<?php echo $this->createUrl("Room/update1",array('id'=>'ID'))?>";
            var tdvalue = element.innerHTML;
            var tdname=element.getAttribute("tdname");
            var data=tdname+'='+tdvalue;
            url1 = url1.replace(/ID/, sid);
            $.ajax({
                //几个参数需要注意一下
                type: "POST",//方法类型
                dataType: "json",//预期服务器返回的数据类型
                url: url1 ,//url
                data:data,
            });
            //当触发时设置父节点的双击事件为ShowElement、也可在这里进行保存动作
            element.setAttribute("ondblclick", "ShowElement(this);");

        }
        //设置该标签的子节点为空
        element.innerHTML = '';
        //添加该标签的子节点，input对象
        element.appendChild(newobj);
        //设置选择文本的内容或设置光标位置（两个参数：start,end；start为开始位置，end为结束位置；如果开始位置和结束位置相同则就是光标位置）
        newobj.setSelectionRange(0, oldhtml.length);
        //设置获得光标
        newobj.focus();

        //设置父节点的双击事件为空
        newobj.parentNode.setAttribute("ondblclick", "");

    }
</script>

```

RoomController.php

```php

<?php

class RoomController extends BaseController {

    protected $model = '';

    public function init() {
        $this->model = substr(__CLASS__, 0, -10);
        parent::init();
        //dump(Yii::app()->request->isPostRequest);
    }

     public function actionIndex( $id="0") {
        set_cookie('_currentUrl_', Yii::app()->request->url);
        $modelName = $this->model;
        $model = $modelName::model();
        if($id!="0")
        {
            $a = ClubNews::model()->find('id='.$id);
            $model->course = $a->name;  // 接收传来的课程名称
            $model->courseid = $a->id;   //接收传来的课程id

        }
        $criteria = new CDbCriteria;
        $data["id"]=$id;
        $criteria->condition=get_where('1=1',$model->courseid,'courseid',$model->courseid,'"');


        //1=1 and workyear="2020-2021" and workterm="上学期" and workcourse="程序设计基础" and workteacher="曾锡山"
        //put_msg("21"." ".$criteria->condition);
        /*criteria为筛选条件，更改对条件即可完成筛选，第一个不用改，第二个改成index里面对应命名
        （即参数，应设置为默认0），第三个为此模块中的筛选的表名，第四个为index里面对应命名（即参数）*/
        parent::_list($model, $criteria, 'index', $data); //调用S
    }

   public function actionCreate() {
       //跨页面传id，从courseinfo->coursework
        /*$a = courseinfo::model()->find('id='.$id);*/
        $modelName = $this->model;
        $model = new $modelName('create');
        $data = array();
        parent::_create($model, 'update', $data,get_cookie('_currentUrl_'));
    }

    public function actionUpdate($id) {
        $modelName = $this->model;
        $model = $this->loadModel($id, $modelName);
        if (!Yii::app()->request->isPostRequest) {
           $data = array();
           $data['model'] = $model;
           $this->render('update', $data);
        } else {
            put_msg($_POST);
            $temp=$_POST[$modelName];
           $this-> saveData($model,$temp);
        }
    }

    public function actionUpdate1($id) {
        $modelName = $this->model;
        $model = $this->loadModel($id, $modelName);
        if(isset($_POST))
        {
            foreach($_POST as $key=>$value)
            {
                $model->{$key}=$value;
            }
            $model->save();
        }
    }


    public function actionDelete($id) {
        parent::_clear($id);
    }
    
    function saveData($model,$post) {
           $model->attributes =$post;
           show_status($model->save(),'保存成功', get_cookie('_currentUrl_'),'保存失败');  
     }

     private function DeleteImage($id)
    {
        $imagePath=ROOT_PATH.'/uploads/image/column/';
        $array = explode(",", $id);
        foreach ($array as $v) {
          $model=NewsColumn::model()->find('id='.$v); 
          if($model->image!=''&&file_exists($imagePath.$model->image))
          {
            unlink($imagePath.$model->image);
          }
        }
        
    }

}


```

models/Room.php

```php
<?php


class Room extends BaseModel {

    public function tableName() {
        return '{{room}}';
    }

    /**
     * 模型验证规则
     */
    public function rules() {
      
        return array(
            array('people', 'required', 'message' => '{attribute} 不能为空'),
			array($this->safeField(),'safe'),
		);
    }	

    /**
     * 模型关联规则
     */
    public function relations() {
        return array(
        );
    }

    /**
     * 属性标签
     */
    public function attributeLabels() {
        return array(
			'id'=>'序号',
			'room'=>'房间号',
			'people'=>'房间人数',
            'courseid' => '课程id',
            'hotel'=>'酒店',
            'room_id'=>'房间内部编号',
            'course' => '课程名称',
 );
}

    /**
     * Returns the static model of the specified AR class.
     */
    public static function model($className = __CLASS__) {
        return parent::model($className);
    }

    public function getCode() {
        return $this->findAll('1=1');
    }

}

```

#### 子功能四：分组算法

SignListController.php

```php
     function actionDGroup($isRoom,$isCar,$courseid,$isMix) { //分组时，男女混合，两两取值
        //$criteria =array();
        //array_push($criteria,$this->createcriteria(1),$this->createcriteria(2));    //下标0是男筛选，1是女筛选
        //$criteria ->params = array (':status' => $你的变量) ;  
        $this->actionDeleteGroup($courseid);
        $group=Group::model()->findALL('courseid = '.$courseid); //查找指定课程的设置的小组（可传递参数）
            foreach($group as $v)
            {
                $p_num=$v->p_num;
                if($isMix == 1)
                    $criteria=$this->createMixSexGroupcriteria($p_num,$courseid);
                else   
                    $criteria=$this->createSameSexGroupcriteria($p_num,$courseid);
                $result = SignList::model()->findAll($criteria);
                $sex='';
                if(!$isMix)
                {
                    foreach($result as $j)
                    {
                        $sex?$sex&=$j->sex:$sex=$j->sex;
                        if($sex)
                        {
                           $j->groupnum=$v->g_id;
                            $j->teacher=$v->teacher;
                            $j->save(); 
                        } 
                        else continue 2;
                    }

                }
                else
                {
                    foreach($result as $j)
                    {
                       $j->groupnum=$v->g_id;
                        $j->teacher=$v->teacher;
                        $j->save(); 
                    }
                }
                }
        if($isRoom == 1)
            $this->actionDroom($courseid);
        if($isCar == 1)
            $this->actionDCar($courseid);
        echo "<script>alert('分组成功!');window.location.href='".$this->createUrl("SignList/result",array("id"=>$courseid,"news_type"=>1))."';</script>";
        //弹窗
     }


    function actionDroom($courseid) {
        //$criteria =array();
        //array_push($criteria,$this->createcriteria(1),$this->createcriteria(2));    //下标0是男筛选，1是女筛选
        //$criteria ->params = array (':status' => $你的变量) ;  
        $room=Room::model()->findALL('courseid = '.$courseid); //查找指定课程的设置的小组（可传递参数）
        foreach($room as $v)
        {
            $p_num=$v->people;
            $criteria=$this->createRoomcriteria($p_num,$courseid); //3
            $result = SignList::model()->findAll($criteria);
            $sex='';    //判断男女是否混住
            foreach($result as $j)
            {
                $sex?$sex&=$j->sex:$sex=$j->sex; //判断男女是否混住
                if($sex)
                {
                   $j->room_id=$v->room_id;
                  // $j->room->$v->room;
                    $j->save(); 
                }
                else continue 2;
            }
        }
        //以下为自动匹配房号
        $rooms = Room::model()->findAll('courseid = '.$courseid);
        $students = SignList::model()->findAll('courseid ='.$courseid.' and room_id is not NULL');
        foreach($rooms as $v)
        {
            foreach($students as $j)
            {
                if($j->room_id == $v->room_id)
                {
                    $j->room = $v->room;
                    $j->save();
                }
            }
        }
        //$this->actionDCar();
     }

     function actionDCar($courseid) { //分组时，男女混合，两两取值
        //$criteria =array();
        //array_push($criteria,$this->createcriteria(1),$this->createcriteria(2));    //下标0是男筛选，1是女筛选
        //$criteria ->params = array (':status' => $你的变量) ;  
        $car=Car::model()->findALL('courseid = '.$courseid); //查找指定课程的设置的小组（可传递参数）
        foreach($car as $v)
        {
            $p_num=$v->car_people;
            $criteria=$this->createCarcriteria($p_num,$courseid);
            $result = SignList::model()->findAll($criteria);
            foreach($result as $j)
            {
                $j->car_id=$v->car_id;
               // $j->car=$v->car;
                $j->save();
            }
        }
        //以下为自动匹配车牌
        $cars = Car::model()->findAll('courseid = '.$courseid);
        $students = SignList::model()->findAll('courseid ='.$courseid.' and car_id is not NULL');
        foreach($cars as $v)
        {
            foreach($students as $j)
            {
                if($j->car_id == $v->car_id)
                {
                    $j->car = $v->car;
                    $j->save();
                }
            }
        }
     }

     function actionDeleteGroup($courseid) { 
         
        $group=Group::model()->findALL('courseid = '.$courseid); //查找指定课程的设置的小组（可传递参数）
        foreach($group as $v)
        {
            $p_num=$v->p_num;
            $criteria=$this->createDeletecriteria($p_num,$courseid);
            $result = SignList::model()->findAll($criteria);
            foreach($result as $j)
            {
                $j->groupnum=NULL;
                $j->room='';
                $j->room_id=NULL;
                $j->car='';
                $j->car_id=NULL;
                $j->teacher='';
                $j->save();
            }
        }
        //echo "<script>alert('分组成功!');location.href='".$_SERVER["HTTP_REFERER"]."';</script>";
     }

     function createDeletecriteria($limit,$courseid)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is not NULL');    //筛选还没被分组的名单，即字段group=0
        $criteria -> condition = get_where($criteria -> condition,$courseid,'courseid',$courseid);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        //$criteria -> order = 'grade asc,class asc';   //指定排序
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }

     function createMixSexGroupcriteria($limit,$courseid)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is NULL');    //筛选还没被分组的名单，即字段group=0
        $criteria -> condition = get_where($criteria -> condition,$courseid,'courseid',$courseid);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        $criteria -> order = 'grade asc,class asc';   //指定排序
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }

     function createSameSexGroupcriteria($limit,$courseid)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is NULL');    //筛选还没被分组的名单，即字段group=0
        $criteria -> condition = get_where($criteria -> condition,$courseid,'courseid',$courseid);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        $criteria -> order = 'sex asc,grade asc,class asc';   //指定排序
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }

     function createRoomcriteria($limit,$courseid)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is not NULL and room_id is NULL');    //筛选还被分组的名单，即字段group不为空
        $criteria -> condition = get_where($criteria -> condition,$courseid,'courseid',$courseid);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        $criteria -> order = 'sex asc,groupnum asc';   //指定排序
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }

     function createCarcriteria($limit,$courseid)
     {
        $criteria = new CDbCriteria() ;  //新建筛选
        //$criteria -> select = array('id','userid','grade','class','status','group');   //筛选字段       
        $criteria -> condition = ('1=1 and groupnum is not NULL and car_id is NULL');    //筛选还被分组的名单，即字段group不为空
        $criteria -> condition = get_where($criteria -> condition,$courseid,'courseid',$courseid);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        //$criteria -> condition = get_where($criteria -> condition,$sex,'sex',$sex,); //筛选性别
        $criteria -> order = 'groupnum asc';   //指定排序
        $criteria -> limit = $limit;  //每次筛选几个
        return  $criteria;
     }
```



**分小组结果页面** (SignList/result.php)

```php+HTML
<div class="box">
    <?php switch("$news_type")
    {
        case 1:echo show_title($this,'分小组结果',0);break;
        case 2:echo show_title($this,'分房结果',0);break;
        case 3:echo show_title($this,'分车结果',0);break;
    }
    ?>
    
    <div class="box-detail">
        <a class="btn" href="<?php echo $this->createUrl('SignList/grouplist');?>"><i class="fa fa-reply"></i>返回</a>
        <a class="btn" href="<?php echo $this->createUrl('SignList/Excel',array('id'=>$id,'name'=>$name));?>"><i class="fa fa-download"></i>导出总表</a>
        <div class="box-detail-bd">
            <h1 align="center">课程：<?php echo $name ?></h1>
            <div class="box-detail-tab box-detail-tab mt15">

            <ul class="flexbox">
                <?php $action=$_REQUEST['news_type'];?>
                <li<?php if($action==1){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('SignList/result&news_type=1',array('id'=>$id));?>">分小组</a>
                </li>
                <li<?php if($action==2){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('SignList/result&news_type=2',array('id'=>$id));?>">分房</a>
                </li>
                <li<?php if($action==3){?> class="current"<?php }?>>
                    <a href="<?php echo $this->createUrl('SignList/result&news_type=3',array('id'=>$id));?>">分车</a>
                </li>
            </ul>
            </div><!--box-detail-tab end-->
            <h2 align="center">最后两列双击即可微调该表格</h2>
        </div>

        <div class="box-table">
        <table class="list">
        <thead>

            <tr>
                <th style='text-align: center;'>编号</th>
                <th style='text-align: center;'>用户姓名</th>
                <th style='text-align: center;'>性别</th>
                <th style='text-align: center;'>年级</th>
                <th style='text-align: center;'>班级</th>
                <?php switch("$news_type")
                {
                case 1:echo "<th style='text-align: center;'>小组号</th>";
                       echo "<th style='text-align: center;'>导师</th>";break;
                case 2:echo "<th style='text-align: center;'>房间编号</th>";
                       echo "<th style='text-align: center;'>实际房间号</th>";break;
                case 3:echo "<th style='text-align: center;'>车编号</th>";
                       echo "<th style='text-align: center;'>实际车牌号</th>";break;
                }
                ?>
            </tr>
        </thead>
                <tbody>

        <?php 
        $index = 1;
        foreach($arclist as $v){ 
        ?>
        <tr>
             <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
           <td style='text-align: center;'><?php echo $v->username; ?></td>
           <td style='text-align: center;'><?php echo $v->sex==1?"男":"女"; ?></td>
           <td style='text-align: center;'><?php echo $v->grade; ?></td>
           <td style='text-align: center;'><?php echo $v->class; ?></td>
           <?php switch("$news_type")
                {
                case 1:echo "<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='groupnum' thid=".$v->id.">".$v->groupnum."</td>";
                       echo "<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='teacher' thid=".$v->id.">".$v->teacher."</td>";break;
                case 2:echo "<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='room_id' thid=".$v->id.">".$v->room_id."</td>";
                       echo "<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='room' thid=".$v->id.">".$v->room."</td>";break;
                case 3:echo "<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='car_id' thid=".$v->id.">".$v->car_id."</td>";
                       echo "<td style='text-align: center;' ondblclick='ShowElement(this)' tdname='car' thid=".$v->id.">".$v->car."</td>";break;
                default:"";
                }
            ?>
        </tr>
        <?php $index++; } ?>
                        </tbody>
                    </table>
                </div><!--box-table end-->
                <div class="box-page c"><?php $this->page($pages);?></div>
            </div><!--box-content end-->
        </div><!--box end-->
    </div>
</div>


<script type="text/javascript">
    function ShowElement(element) {
        var oldhtml = element.innerHTML;
        //创建新的input元素
        var newobj = document.createElement('input');
        //为新增元素添加类型
        newobj.type = 'text';
        //为新增元素添加value值
        newobj.value = oldhtml;
        //为新增元素添加光标离开事件
        newobj.onblur = function() {
            //当触发时判断新增元素值是否为空，为空则不修改，并返回原有值 
            element.innerHTML = this.value == oldhtml ? oldhtml : this.value;
            var sid= element.getAttribute("thid");
            var url1 = "<?php echo $this->createUrl("SignList/update1",array('id'=>'ID'))?>";
            var tdvalue = element.innerHTML;
            var tdname=element.getAttribute("tdname");
            var data=tdname+'='+tdvalue;
            url1 = url1.replace(/ID/, sid);
            $.ajax({
                //几个参数需要注意一下
                type: "POST",//方法类型
                dataType: "json",//预期服务器返回的数据类型
                url: url1 ,//url
                data:data,
            });
            //当触发时设置父节点的双击事件为ShowElement、也可在这里进行保存动作
            element.setAttribute("ondblclick", "ShowElement(this);");

        }
        //设置该标签的子节点为空
        element.innerHTML = '';
        //添加该标签的子节点，input对象
        element.appendChild(newobj);
        //设置选择文本的内容或设置光标位置（两个参数：start,end；start为开始位置，end为结束位置；如果开始位置和结束位置相同则就是光标位置）
        newobj.setSelectionRange(0, oldhtml.length);
        //设置获得光标
        newobj.focus();

        //设置父节点的双击事件为空
        newobj.parentNode.setAttribute("ondblclick", "");

    }
</script>
```

#### 子功能五：导出excel表格

![](https://s4.ax1x.com/2022/01/08/7ilxSK.jpg)

SignListController.php

```php
   public function actionExcel($id,$name){
        //先定义一个excel文件
        $filename   =   date('【分组结果总表】('.date('Y-m-d H:i:s').'导出)').".xls";
        header("Content-Type: application/vnd.ms-execl");
        header("Content-Type: application/vnd.ms-excel; charset=utf-8");
        header("Content-Disposition: attachment; filename=$filename");
        header("Pragma: no-cache");
        header("Expires: 0");
        //条件
        $criteria = new CDbCriteria() ;  //新建筛选
        $criteria -> select = array('id','userid','username','sex','grade','class','status','teacher','groupnum','car_id','car','room_id','room');   //筛选字段       
        $criteria -> condition = ('1=1');    //筛选还没被分组的名单，即字段group=0
        $criteria -> condition = get_where($criteria -> condition,$id,'courseid',$id);  //筛选课程（可传递参数） 
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        $criteria -> order = 'groupnum asc,grade asc,class asc';   //指定排序


        
        //使用html语句生成显示的格式
        $excel_content      =   '<meta http-equiv="content-type" content="application/ms-excel; charset=utf-8"/>';
        $excel_content     .=   '<table border="1" style="font-size:14px;">';
        $excel_content     .=   '<h1 align="center">课程：'.$name.'</h1>';
        $excel_content     .=   '<thead>
                                     <tr>
                                        <th>用户姓名</th>
                                        <th>性别</th>
                                        <th>年级</th>
                                        <th>班级</th>
                                        <th>小组号</th>
                                        <th>导师</th>
                                        <th>房间编号</th>
                                        <th>实际房间号</th>
                                        <th>车编号</th>
                                        <th>实际车牌号</th>
                                     </tr>
                                </thead>
                                 ';
        //查找最新的固化数据
        $search = SignList::model()->findAll($criteria);

        //计算每个小组的人数
        $criteria1 = new CDbCriteria() ;  //新建筛选
        $criteria1 -> select = array('g_id','p_num');   //筛选字段  
        $criteria -> order = 'g_id asc';     
        $g_num = Group::model()->findALL($criteria1);
        //html语句填充数据
        if(empty($search)){}
        else{
            $i=0;
            $j=0;
            foreach ($search as $k) {
                $excel_content  .= '<td>'.$k->username.'</td>';
                $excel_content  .= '<td>'.($k->sex==1?"男":"女").'</td>';
                $excel_content  .= '<td>'.$k->grade.'</td>';
                $excel_content  .= '<td>'.$k->class.'</td>';
                if($j==0)
                {
                    $temp=count(SignList::model()->findAll("groupnum=".$g_num[$i]->g_id));
                    $excel_content  .= '<td style="text-align: center;" rowspan='.$temp.'>'.$k->groupnum.'</td>';
                    $j++;$i++;
                }
                else if($j<$temp-1) $j++;
                else $j=0;
                $excel_content  .= '<td>'.$k->teacher.'</td>';
                $excel_content  .= '<td>'.$k->room_id.'</td>';
                $excel_content  .= '<td>'.$k->room.'</td>';
                $excel_content  .= '<td>'.$k->car_id.'</td>';
                $excel_content  .= '<td>'.$k->car.'</td></tr>';
            }
        }
        $excel_content  .=  '</table>';
        echo $excel_content;
        die;
}

```

models/importExcel.php

```php
<?php


class ImportExcel
{

    function isEndOf($str,$hzArray=array()){
        foreach ($hzArray as $hz){
            if ($this->file_hz($str)==$hz)return true;
        }
        return false;
    }

    function file_hz($file){
        //put_msg('hzhzhz');
        //put_msg(substr($file, strrpos($file, '.')+1));
        return substr($file, strrpos($file, '.')+1);
    }

    public function import($excelFile = "",$modelName='',$nameArray=array(),$colFirst='B',$rowFirst=3)
    {
        if(!$this->isEndOf($excelFile,array('xls','xlsx')))
            return false;

        Yii::$enableIncludePath = false;
        Yii::import('application.extensions.PHPExcel.PHPExcel', 1);
        $extension=$this->file_hz($excelFile);
        if ($extension=='xls') {
            $excelReader = PHPExcel_IOFactory::createReader('Excel5');
        } else {
            $excelReader = PHPExcel_IOFactory::createReader('Excel2007');
        }

        $objPHPExcel = $excelReader->load($excelFile);
        $sheet = $objPHPExcel->getSheet(0);
        $highestRow = $sheet->getHighestRow(); // 取得总行数
        //put_msg($highestRow);
        for ($row = $rowFirst; $row <= $highestRow; $row++){
            $a = new $modelName();
            for($i=0 ; $i<count($nameArray);$i++)
            {
                $col = chr(ord($colFirst)+$i);
                $a->{$nameArray[$i]}=$sheet->getCell($col.$row)->getValue();
            }
            $a->save();
        }
        return true;
    }
    
}
```

### 需求十：实现时间选择输入框

![](https://s4.ax1x.com/2022/01/11/7e3Nmq.jpg)

实现：在数据库表内数据类型选择datetime

update.php

```php+HTML
                        <td><?php echo $form->labelEx($model, 'start_time'); ?></td>
                        <td>
                            <?php echo $form->textField($model, 'start_time', array('class' => 'input-text')); ?>
                            <?php echo $form->error($model, 'start_time', $htmlOptions = array()); ?>
                        </td>
                        <td><?php echo $form->labelEx($model, 'end_time'); ?></td>
                        <td>
                            <?php echo $form->textField($model, 'end_time', array('class' => 'input-text')); ?>
                            <?php echo $form->error($model, 'end_time', $htmlOptions = array()); ?>
                        </td>
                    </tr>


<script>
    we.tab('.box-detail-tab li','.box-detail-tab-item');
    var club_id=0;
    $('#Advertisement_start_time').on('click', function(){
    WdatePicker({startDate:'%y-%M-%D',dateFmt:'yyyy-MM-dd'});});

    $('#Advertisement_end_time').on('click', function(){
    WdatePicker({startDate:'%y-%M-%D',dateFmt:'yyyy-MM-dd'});});

</script>
```

### 需求十一：省市二级联动

实现选择省份后城市选择框只显示该省份的下一级行政单位

![](https://s4.ax1x.com/2022/01/11/7eUpgU.jpg)

location表：id是城市id  pid是上一级行政单位id  name是城市名称

![](https://s4.ax1x.com/2022/01/11/7eNycD.jpg)

models/Location.php

```php
<?php

use yii\validatoers;

class Location extends BaseModel {

    public function tableName() {
        return '{{location}}';
    }

    /**
     * 模型验证规则
     */
    public function rules() {
      
        return array(
			array('name,pid','safe'), 
		);
    }	


    /**
     * 模型关联规则
     */
    public function relations() {
        return array(
		 
		);
    }

    /**
     * 属性标签
     */
    public function attributeLabels() {
        return array(
			'id'=>'ID',
            'name'=>'名字',
            'level'=>'上级id',
 );
}

    /**
     * Returns the static model of the specified AR class.
     */
    public static function model($className = __CLASS__) {
        return parent::model($className);
    }
	
	

    public function getCode() {
        return $this->findAll('1=1');
    }
}

```

ClubNews/update.php

```php+HTML
                    <tr>
                        <td><?php echo $form->labelEx($model, 'province'); ?></td>
                        <td>
                            <select name="province" id="province">
                                <option value="">请选择</option>
                                <?php foreach ($province as $v) { ?>
                                    <option value="<?php echo $v->id; ?>" <?php if ($model->province == $v->id) { ?> selected<?php } ?>><?php echo $v->name; ?></option>
                                <?php } ?>
                            </select>    
                            <?php echo $form->hiddenField($model, 'province', array('class' => 'input-text')); ?>
                            <?php echo $form->error($model, 'province', $htmlOptions = array()); ?>
                        </td>

                        <td><?php echo $form->labelEx($model, 'city'); ?></td>
                        <td>
                            <select name="city" id='city'>
                                <option value="">请选择</option>
                                <?php if (isset($city)) foreach ($city as $v) { ?>
                                    <option value="<?php echo $v->id; ?>" <?php if ($model->city == $v->id) { ?> selected<?php } ?>><?php echo $v->name; ?></option>
                                <?php } ?>
                            </select>
                            <?php echo $form->hiddenField($model, 'city', array('class' => 'input-text')); ?>
                            <?php echo $form->error($model, 'city', $htmlOptions = array()); ?>
                        </td>
                    </tr>

<script>
    //省市二级联动
    $("#province").change(function() {
        $('#city').html("<option value=''>请选择</option>");
        $('#ClubNews_province').val($("#province").val());
        getLocation("province",'#city');
        $('#ClubNews_province').val($("#province option:selected").text());
        //document.search.submit();
    });

    $("#city").change(function() {
            $('#ClubNews_city').val($("#city option:selected").text());
        });

    function getLocation(sourece,target) {
        var myselect = document.getElementById(sourece);
        var index = myselect.selectedIndex;
        var code = myselect.options[index].value;
        getData(code,target);
    }

    function getData(code, element) {
        $.ajax({
            url: "<?php echo $this->createUrl('select/getLocation'); ?>",
            data: {
                code: code
            },
            type: "get",
            success: function(res) {
                var data = JSON.parse(res);
                var str = "<option value=''>请选择</option>";
                for (var i = 0; i < data.length; i++) {
                    str += "<option value='" + data[i].id + "'>" + data[i].name + "</option>";
                }
                // //把所有<option>放到区的下拉列表里
                $(element).html(str);
            }
        });
    }
</script> 
```

### 需求十二：在另一个页面实时查看有未结束活动的城市

显示哪些城市有未结束的活动并且显示活动的数量

![](https://s4.ax1x.com/2022/01/11/7eaMWV.jpg)

CityController.php

```php
    public function actionIndex() {
    set_cookie('_currentUrl_', Yii::app()->request->url);
    $modelName = "ClubNews";//  使用ClubNews的model
    $model = $modelName::model();
    $criteria = new CDbCriteria;
    $criteria->condition='1=1';
    $criteria->condition=get_where($criteria->condition,'活动已结束','activeStatus!=','活动已结束','"'); //筛选未结束的活动
    $criteria->select = "province,city"; //要查询的字段
    $criteria->group = "province,city"; // 按照省份和城市分组（即筛出不重复的有活动的城市）
    $data = array();
    parent::_list($model, $criteria, 'index', $data,20);
    }
```

City/index.php

```php+HTML
<div class="box-table">
    <table class="list">
<thead>


    <tr>
        <th style='text-align: center;'>编号</th>
        <th style='text-align: center;'>城市</th>
        <th style='text-align: center;'>当前进行活动数目</th>
        <th style='text-align: center;'>当前城市活动详情</th>
    </tr>
</thead>
        <tbody>

<?php 
$index = 1;
foreach($arclist as $v){
 $count=count(ClubNews::model()->FindAll("city='".$v->city."' AND activeStatus!='活动已结束' AND approvalStatus='通过'"));
 //找出ClubNews中对应城市名字相同的且符合条件的名字有多少个活动
?>
<tr>
    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
   <td style='text-align: center;'><?php echo $v->province." ".$v->city?></td>
   <td style='text-align: center;'><?php echo $count; ?></td>
   <td style='text-align: center;'>
        <a class="btn btn-blue" href="<?php echo $this->createUrl('ClubNews/index', array('province'=>$v->province,'city'=>$v->city,'approvalStatus'=>'通过','activeStatus1'=>'活动已结束',"isFake"=>1));?>" title="活动详情"><i class="fa fa-table ">活动详情</i></a>
    </td>
</tr>
<?php $index++; } ?>
                </tbody>
            </table>
        </div><!--box-table end-->
        <div class="box-page c"><?php $this->page($pages);?></div>
    </div><!--box-content end-->
</div><!--box end-->
```

### 需求十三：学生和家长绑定功能

要求实现一个子女可对多个家长 ，一个家长可对多个子女 

使用userinfo表存学生信息 ， user_parent表储存家长的信息

userinfo表的p_id_set存放家长的编号   user_parent 表用k_id_set放子女的信息

userinfo表

![](https://s4.ax1x.com/2022/01/12/7Kll5V.jpg)

models/userinfo.php

```php
  public function getParentCondition()
    {
        $ids=trim(implode(',',explode('|',$this->p_id_set)),','); //用   |  来分割字符串 
        //trim函数 把 , 从字符串中移除
        put_msg($ids); //第一次
        $w = 'id in (' . $ids . ')';   
        put_msg($w); //第二次
        if (!$ids) $w = '0';
        return $w; 
    }

    public function getParent(){
        $w = $this->getParentCondition();
        $tmp = UserParent::model()->findAll($w); //在家长表里面找到子女对应的家长（对象数组）
        return $tmp;  //找到监护人

    }

    protected function afterFind()
    {
        return parent::afterFind();
    }


    public function getParentInfo(){
        $p = $this->getParent();
        $res='';
        if($p){
            foreach($p as $v){
                $res .= ",".$v->name.'('.$v->phone.')';
            }
            put_msg($res); //第三次
            $res = trim($res,',');//把最左边的那个逗号去掉
        }
        return $res;
    }
}
```

第一二三次put_msg的结果

![](https://s4.ax1x.com/2022/01/12/7KlwUx.jpg)

### 需求十四：下拉框筛选填入功能

要求在分组设置的页面用下拉框选择在teaSignList页面报名成功(status=3) 且课程相同的导师

![](https://s4.ax1x.com/2022/01/12/7K1qSO.jpg)

Group/update.php

```php+HTML
                   <?php $Array = teaSignList::model()->findAll("courseid =".$_SESSION["courseid"]." and status = '3'") ;
                    ?>
                     <td ><?php echo $form->labelEx($model, 'teacher'); ?></td>
                    <td>
                        <?php
                                echo Select2::activeDropDownList($model, 'tea_id', Chtml::listData($Array, 'id', 'username'), array('prompt'=>'请选择','style'=>'width:95%;','onchange' =>'selectOnchangapply_title(this)',));
                        //从对象数组Array中取出每个对象里面的id,username是id对应的username显示在下拉框里
                        //用户选择之后对应的id存入tea_id
                        ?>
                                <?php echo $form->error($model, 'tea_id', $htmlOptions = array());?>
                         </td>
```

models/Group.php

```php
    protected function beforeSave() {
        parent::beforeSave();
        $temp=teaSignList::model()->find("id=".$this->tea_id);
        $this->teacher=$temp->username; //找到没对应id的username存入teacher中
        $temp->groupnum=$this->g_id; //把对应老师的g_id回填入teaSignList对应表的groupnum里面
        $temp->save();

         
        // 圖文描述處理
        return true;
    }
```

