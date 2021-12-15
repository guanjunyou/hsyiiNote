## 曾锡山HsYii框架笔记

### 需求一：在ClubNews页面上取出PayType对应的数据库(pay_type)里的缴费类型进行遍历填写并结合ClubNews的项目类型一起传到ClubPaySet对应的数据库(hash)

![]()

**要求实现：**

**1，实施输入实时显示总钱数，并在下一次进入该页面时输入框内数字不发生变化**

**2，pay_type里面有多少个类型在ClubNews/update.php就要显示多少个输入框**

![image-20211213093400146](../../AppData/Roaming/Typora/typora-user-images/image-20211213093400146.png)

![image-20211213093427149](../../AppData/Roaming/Typora/typora-user-images/image-20211213093427149.png)

![image-20211213093459965](../../AppData/Roaming/Typora/typora-user-images/image-20211213093459965.png)

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

![image-20211213093936601](../../AppData/Roaming/Typora/typora-user-images/image-20211213093936601.png)

![image-20211213093956034](../../AppData/Roaming/Typora/typora-user-images/image-20211213093956034.png)

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

![image-20211215145459155](../../AppData/Roaming/Typora/typora-user-images/image-20211215145459155.png)

