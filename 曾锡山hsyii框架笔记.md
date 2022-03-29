

## 曾锡山HsYii框架笔记

**编辑规范：图片均使用图床上的URL  配代码和思路**

[TOC]

### 基本功能：

#### 页面搜索框：

根据时间/地点/学校等关键字搜素字段

ClubNews/index 页面   此页面要结合省市区三级联动

![](https://s1.ax1x.com/2022/03/27/qB5pWj.jpg)

ClubNews/index.php

```php+HTML
<?php
 $years=base_year::model()->findALL();
 $terms=base_term::model()->findALL();
?>
<div class="box">

     <?php if(isset($_REQUEST['isFake'])) {; ?>
        <?php echo show_title($this,"活动详情",1);?>
    <?php }else{echo show_title($this);}?>

    
    <div class="box-content">
        <div class="box-header">
            <?php if(!isset($_REQUEST['isFake'])) {?><a class="btn" href="<?php echo $this->createUrl('create');?>"><i class="fa fa-plus"></i>添加</a><?php }?>
            <a class="btn" href="javascript:;" onclick="we.reload();"><i class="fa fa-refresh"></i>刷新</a>
            <a style="display:none;" id="j-delete" class="btn" href="javascript:;" onclick="we.dele(we.checkval('.check-item input:checked'), deleteUrl);"><i class="fa fa-trash-o"></i>刪除</a>
        </div><!--box-header end-->

     <form action="<?php echo Yii::app()->request->url;?>" method="get">
    <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">
    <input type="hidden" name="approvalStatus" value="<?php echo Yii::app()->request->getParam('approvalStatus');?>">
    <input type="hidden" name="activeStatus" value="<?php echo Yii::app()->request->getParam('activeStatus');?>">
    <input type="hidden" name="activeStatus1" value="<?php echo Yii::app()->request->getParam('activeStatus1');?>">


    <label style="margin-right:10px;">
    <span>活动时间查询：</span>
    <input style="width:120px;" class="input-text date" placeholder="活动开始日期" type="text" id="start_date" name="cstart_date" value="<?php echo Yii::app()->request->getParam('start_date');?>">
    <span>-</span>
    <input style="width:120px;" class="input-text date" placeholder="活动结束日期" type="text" id="end_date" name="cend_date" value="<?php echo Yii::app()->request->getParam('end_date');?>">
    </label>

    <label style="margin-right:10px;">
    <span>报名时间查询：</span>
    <input style="width:120px;" class="input-text date" placeholder="报名开始日期" type="text" id="start_date" name="start_date" value="<?php echo Yii::app()->request->getParam('start_date');?>">
    <span>-</span>
    <input style="width:120px;" class="input-text date" placeholder="报名结束日期" type="text" id="end_date" name="end_date" value="<?php echo Yii::app()->request->getParam('end_date');?>">
    </label>
    <label style="margin-right:10px;">
    <span>省：</span>
    <select name="province" id="province">
        <option value="">请选择</option>
        <?php foreach ($province as $v) { ?>
            <option value="<?php echo $v->id; ?>" <?php if (Yii::app()->request->getParam('province') == $v->id) { ?> selected<?php } ?>><?php echo $v->name; ?></option>
        <?php } ?>
    </select>
    </label>
    <label style="margin-right:10px;">
    <span>市：</span>
    <select name="city" id='city'>
        <option value="">请选择</option>
        <?php if(isset($city)) foreach ($city as $v) { ?>
            <option value="<?php echo $v->id; ?>" <?php if (Yii::app()->request->getParam('city') == $v->id) { ?> selected<?php } ?>><?php echo $v->name; ?></option>
        <?php } ?>
    </select>
    </label>
    <label style="margin-right:10px;">
     <span>镇区：</span>
     <select name="district" id='district'>
         <option value="">请选择</option>
         <?php if(isset($district)) foreach ($district as $v) { ?>
             <option value="<?php echo $v->id; ?>" <?php if (Yii::app()->request->getParam('district') == $v->id) { ?> selected<?php } ?>><?php echo $v->name; ?></option>
         <?php } ?>
     </select>
    </label>

    <div>
     <label style="margin-right:10px;">
         <span>根据学校/活动关键字查询：</span>
         <input style="width:200px;" class="input-text" type="text" name="keywords" value="<?php echo Yii::app()->request->getParam('keywords');?>">
     </label>
        <button class="btn btn-blue" type="submit" array=>查询</button>
    </div>


    </form>
</div><!--box-search end-->

```

js  script

```js

$("#province").change(function() {
        $('#city').html("<option value=''>请选择</option>");
        getLocation("province",'#city');
        //document.search.submit();
    });
$("#city").change(function() {
    $('#district').html("<option value=''>请选择</option>");
    getLocation("city",'#district');
    //document.search.submit();
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



var $start_date=$('.date');
var $end_date=$('.date');  //把时间初始化默认为今天
$start_date.on('click', function(){
    WdatePicker({startDate:'%y-%M-%D',dateFmt:'yyyy-MM-dd'});
});
$end_date.on('click', function(){
    WdatePicker({startDate:'%y-%M-%D',dateFmt:'yyyy-MM-dd'});
});

```

ClubNewsController.php  注意形参

```php
    public function actionIndex($district='',$city="",$styear="",$sterm="",$cstart_date="",$cend_date="",$start_date="",$end_date="",$approvalStatus="",$activeStatus="",$activeStatus1="",$province="",$keywords='') {
        //put_msg($approvalStatus);
    set_cookie('_currentUrl_', Yii::app()->request->url);
    $modelName = $this->model;
    $model = $modelName::model();
    $criteria = new CDbCriteria;
    $w1=get_where('1=1',$styear,'f_year',$styear,'"');
    $criteria->order = 'id desc';
    $criteria->condition=get_where($w1,$sterm,'f_term',$sterm,'"');
    $criteria->condition=get_where($criteria->condition,$cstart_date,'signIn_date_start>=',$cstart_date,'"');
    $criteria->condition=get_where($criteria->condition,$cend_date,'signIn_date_end<=',$cend_date,'"');
    $criteria->condition=get_where($criteria->condition,$start_date,'sign_date_start>=',$start_date,'"');
    $criteria->condition=get_where($criteria->condition,$end_date,'sign_date_end<=',$end_date,'"');
    if($approvalStatus=='保存') $criteria->condition=get_where_in($criteria->condition,"'保存','提交'",'approvalStatus',"'保存','提交'",'"');
    else $criteria->condition=get_where($criteria->condition,$approvalStatus,'approvalStatus',$approvalStatus,'"');
    $criteria->condition=get_where($criteria->condition,$activeStatus,'activeStatus',$activeStatus,'"');
    $criteria->condition=get_where($criteria->condition,$activeStatus1,'activeStatus!=',$activeStatus1,'"');
    $criteria->condition=get_like($criteria->condition,'news_club_name,name',$keywords,'');
    if($_SESSION['F_ROLENAME']!=='后台超级管理员' && $_SESSION['F_ROLENAME']!=='省级教育主管部门' && $_SESSION['F_ROLENAME']!=='市级教育主管部门')
        $criteria->condition=get_where($criteria->condition,$_SESSION['adminid'],'updateuserid',$_SESSION['adminid'],'"');
    if($_SESSION['F_ROLENAME']==='省级教育主管部门')
        $criteria->condition=get_where($criteria->condition,$_SESSION['F_province'],'province',$_SESSION['F_province'],'"');
    if($_SESSION['F_ROLENAME']==='市级教育主管部门')
        $criteria->condition=get_where($criteria->condition,$_SESSION['F_city'],'city',$_SESSION['F_city'],'"');
    if($_SESSION['F_ROLENAME']==='县级教育主管部门')
        $criteria->condition=get_where($criteria->condition,$_SESSION['F_district'],'district',$_SESSION['F_district'],'"');
        $data = array();
    if (is_numeric($province)) {
        $p=location::model()->find('id='.$province)->name;
        $criteria->condition=get_where($criteria->condition,$p,'province',$p,'"');
        $data['city'] = Location::model()->findAll('pid=' . $province);
    }
    else{
        $criteria->condition=get_where($criteria->condition,$province,'province',$province,'"');
    }
    if (is_numeric($city)) {
        $c=location::model()->find('id='.$city)->name;
        $criteria->condition=get_where($criteria->condition,$c,'city',$c,'"');
        $data['district'] = Location::model()->findAll('pid=' . $city);
    }
    else{
        $criteria->condition=get_where($criteria->condition,$city,'city',$city,'"');
    }
    if (is_numeric($district)) {
        $d=location::model()->find('id='.$district)->name;
        $criteria->condition=get_where($criteria->condition,$d,'district',$d,'"');
    }
    else{
        $criteria->condition=get_where($criteria->condition,$district,'district',$district,'"');
    }
       
        $data['province'] = Location::model()->findAll('pid=0');
        //put_msg($criteria->condition);
    parent::_list($model, $criteria, 'index', $data,20);
    }
```

#### 联动选择框



![](https://s1.ax1x.com/2022/03/28/qDlPo9.jpg)

ClubNews/update.php

```php+HTML
                    <?php if($_SESSION['F_ROLENAME']==='后台超级管理员' || $_SESSION['F_ROLENAME']==='市级教育主管部门' || $_SESSION['F_ROLENAME']==='省级教育主管部门' || $_SESSION['F_ROLENAME']==='学校' || $_SESSION['F_ROLENAME']==='县级教育主管部门'){?>
                    <tr>
                        <td><?php echo $form->labelEx($model, 'serviceClub');?></td>
                        <td>
                            <div id="bindService"><?php echo $form->hiddenField($model, 'serviceClub', array('class' => 'input-text')); ?></div>
                            <?php echo $form->error($model, 'serviceClub', $htmlOptions = array()); ?>
                        </td>
                        <td><?php echo $form->labelEx($model, 'baseClub');?></td>
                        <td>
                            <div id="bindBase"><?php echo $form->hiddenField($model, 'baseClub', array('class' => 'input-text')); ?></div>
                            <?php echo $form->error($model, 'baseClub', $htmlOptions = array()); ?>
                        </td>
                    </tr>
                    <?php } else {?>
                        <tr>
                            <td><?php echo $form->labelEx($model, 'serviceClub');?></td>
                            <td>
                                <?php echo $_SESSION['TUNIT']?>
                                <?php echo $form->hiddenField($model,'serviceClub', array('value' => $_SESSION['TUNIT_id'])); ?>
                                <?php echo $form->error($model, 'serviceClub', $htmlOptions = array()); ?>
                            </td>
                            <td><?php echo $form->labelEx($model, 'baseClub');?></td>
                            <td>
                                <div id="bindBase"><?php echo $form->hiddenField($model, 'baseClub', array('class' => 'input-text')); ?></div>
                                <?php echo $form->error($model, 'baseClub', $htmlOptions = array()); ?>
                            </td>
                        </tr>
                    <?php }?>
```

页面 js

```js
 //服务商绑定
    $.ajax({
        type: 'get',
        url: '<?php echo $this->createUrl("ClubList/Servicelist");?>',
        data: {},
        dataType: "json",
        success: function (res) {
            var data = [];
            console.log(res);
            for (var i = 0; i < res.data.length; i++) {
                data.push({name: res.data[i].club_name, value: res.data[i].id});
            }
            xmSelect.render({
                el: '#bindService',
                theme: {
                    color: '#368ee0',
                },
                initValue: [<?php echo $model->serviceClub?>],
                autoRow: true,
                direction: 'up',
                language: 'zn',
                filterable: true,
                radio: true,
                clickClose: true,
                <?php if(!empty($_REQUEST['approvalStatus'])&&$_REQUEST['approvalStatus']!=='保存'){?>
                disabled:true,
                <?php }?>
                model: {
                    label: {
                        type: 'text',
                        text: {
                            //左边拼接的字符
                            left: '',
                            //右边拼接的字符
                            right: '',
                            //中间的分隔符
                            separator: ', ',
                        },
                    }
                },
                data: data,
                on: function (data) {
                    var selectArr = data.arr;
                    var seachArr = [];
                    for (var j = 0; j < selectArr.length; j++) {
                        seachArr.push(selectArr[j].value)
                    }
                    $("#ClubNews_serviceClub").val(seachArr.toString());
                    basebind();
                }
            })
        },
        error: function (err) {
            console.log(err)
        }
    });

    var service=document.getElementById("ClubNews_serviceClub");
    <?php if($model->serviceClub) {?> basebind();<?php }else{?>
    if(service!=null) basebind();
    <?php }?>

    //研学基地绑定
    function basebind(){


        var svalue=service.value;
        var URL = '<?php echo $this->createUrl("ClubList/Baselist",array('sid'=>'ID'));?>'
        URL = URL.replace(/ID/, svalue); /*替换ID为选中的记录的id*/
        $.ajax({
            type: 'get',
            url: URL,
            data: {},
            dataType: "json",
            success: function (res) {
                var data = [];
                console.log(res);
                for (var i = 0; i < res.data.length; i++) {
                    data.push({name: res.data[i].club_name, value: res.data[i].id});
                }
                xmSelect.render({
                    el: '#bindBase',
                    theme: {
                        color: '#368ee0',
                    },
                    <?php if(!empty($_REQUEST['approvalStatus'])&&$_REQUEST['approvalStatus']!=='保存'){?>
                    disabled:true,
                    <?php }?>
                    initValue: [<?php echo $model->baseClub?>],
                    autoRow: true,
                    direction: 'up',
                    language: 'zn',
                    filterable: true,
                    data: data,
                    on: function (data) {
                        var selectArr = data.arr;
                        var seachArr = [];
                        for (var j = 0; j < selectArr.length; j++) {
                            seachArr.push(selectArr[j].value)
                        }
                        $("#ClubNews_baseClub").val(seachArr.toString());
                    }
                })
            },
            error: function (err) {
                console.log(err)
            }
        });
    }

```

ClubListController.php

```php
    public function actionBaselist($sid='')
    {
        $s = 'id,club_name';
        if(empty($sid)){
            $rs=array('data'=>'');
            Basefun::model()->echoEncode($rs);
        }
        else{
            $tmp = ClubList::model()->find("id=".$sid);
            $bid=$tmp->clubBind;
            if(empty($bid)){
                $rs=array('data'=>'');
                Basefun::model()->echoEncode($rs);
            }
            else{
                $criteria = new CDbCriteria;
                $criteria->condition=get_where('1=1',"研学基地",'sort',"研学基地",'"');
                $criteria->condition=get_where_in($criteria->condition,$bid,'id',$bid,'"');
                $da =ClubList::model()->recToArray($criteria,$s);
                $rs=array('data'=>$da);
                Basefun::model()->echoEncode($rs);
            }
        }
    }

    public function actionServicelist()
    {
        $s = 'id,club_name';
        $criteria = new CDbCriteria;
        $criteria->condition=get_where('1=1',"服务机构",'sort',"服务机构",'"');
        $da =ClubList::model()->recToArray($criteria,$s);
        $rs=array('data'=>$da);
        Basefun::model()->echoEncode($rs);
    }

```

#### 下拉框：

![](https://s1.ax1x.com/2022/03/28/qD1IEQ.jpg)

ClubNews/update.php

select 第一个 typeid 是本页面表里面字段 , 第二个的id 是 ActivityType 里面的课程类型id  type 是ActivityType里面的课程类型名称

```php+HTML
                           <?php $cds=ActivityType::model()->findAll();
                                echo Select2::activeDropDownList($model, 'typeid', Chtml::listData($cds, 'id', 'type'), array('prompt'=>'请选择','style'=>'width:95%;')); ?>
                                <?php echo $form->error($model, 'typeid', $htmlOptions = array());?>
                         </td>
```

#### 弹出选框



![](https://s1.ax1x.com/2022/03/28/qD3N5j.jpg)

InfoDiffusion/update.php

```php+HTML
                    <?php if($_SESSION['F_ROLENAME']=="后台超级管理员" ||$_SESSION['F_ROLENAME']=="市级教育主管部门" ||$_SESSION['F_ROLENAME']=="省级教育主管部门"){?>

                        <!--下面是选择机构-->
                        <td style="padding:10px;"><?php echo $form->labelEx($model, 'club_id'); ?></td>
                        <td >
                            <?php echo $form->hiddenField($model, 'club_id', array('class' => 'input-text')); ?>
                            <span id="club_box"><?php if($model->news_club_name!=null){?><span class="label-box"><?php echo $model->news_club_name;?></span><?php }?></span>
                            <?php if(!isset($_REQUEST['approvalStatus'])|| $_REQUEST['approvalStatus']==="保存" ){ ?>
                            <input id="club_select_btn" class="btn" type="button" value="选择">
                            <?php }?>
                            <?php echo $form->error($model, 'club_id', $htmlOptions = array()); ?>
                        </td>

                    <?php } else {?>
                        <!--下面是选择机构-->
                        <td style="padding:10px;"><?php echo $form->labelEx($model, 'club_id'); ?></td>
                        <td >
                            <?php echo $form->hiddenField($model, 'club_id', array('value'=>$_SESSION['TUNIT_id'],'class' => 'input-text')); ?>
                            <span id="club_box"><?php if($_SESSION['TUNIT_id']!=null){?><span class="label-box"><?php echo $_SESSION['TUNIT'];?></span><?php }?></span>
                            <?php echo $form->error($model, 'club_id', $htmlOptions = array()); ?>
                        </td>
                        </tr>                       
                    <?php }?>
```

js 代码

```js
    // 選擇單位
    var $club_box=$('#club_box');
    var $InfoDiffusion_club_id=$('#infoDiffusion_club_id');
    $('#club_select_btn').on('click', function(){
        $.dialog.data('club_id', 0);
        $.dialog.open('<?php echo $this->createUrl("select/club", array('partnership_type'=>16));?>',{
            id:'danwei',
            lock:true,
            opacity:0.3,
            title:'选择具体内容',
            width:'500px',
            height:'60%',
            close: function () {
                //console.log($.dialog.data('club_id'));
                if($.dialog.data('club_id')>0){
                    club_id=$.dialog.data('club_id');
                    $InfoDiffusion_club_id.val($.dialog.data('club_id')).trigger('blur');
                    $club_box.html('<span class="label-box">'+$.dialog.data('club_title')+'</span>');
                }
            }
        });
    });

```

select/club.php

```html
<div class="box">
    <div class="box-content">
        <div>
            <form action="<?php echo Yii::app()->request->url;?>" method="get">
                <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">
                <label style="margin-right:10px;">
                    <span>关键字：</span>
                    <input id="club" style="width:200px;" class="input-text" type="text" name="keywords" value="<?php echo Yii::app()->request->getParam('keywords');?>">
                </label>
                <button class="btn btn-blue" type="submit">查询</button>
            </form>
        </div><!--box-search end-->
        <div class="box-table">
            <table class="list">
                <thead><tr><th>点击选择</th></tr></thead>
                <tbody>
<?php foreach($arclist as $v){ ?>
    <tr data-id="<?php echo $v->select_id; ?>" data-code="<?php echo $v->select_code; ?>" data-title="<?php echo $v->select_title; ?>">
        <td><?php echo $v->select_title; ?></td>
    </tr>
<?php } ?>
                </tbody>
            </table>
        </div><!--box-table end-->
        <div class="box-page c"><?php $this->page($pages); ?></div>
    </div><!--box-content end-->
</div><!--box end-->
<script>
$(function(){
    api = $.dialog.open.api;	// 			art.dialog.open扩展方法
    if (!api) return;

    // 操作对话框
    api.button( { name: '取消' } );
    $('.box-table tbody tr').on('click', function(){
    //    var id=$(this).attr('data-id');
    //    var title=$(this).attr('data-title');
        $.dialog.data('club_id', $(this).attr('data-id'));
        $.dialog.data('club_code', $(this).attr('data-code'));
        $.dialog.data('club_title', $(this).attr('data-title'));
        $.dialog.close();
    });
});
</script>
```

selectController.php

```php
   public function actionClub($keywords = '', $partnership_type = 0, $project_id = 0, $no_cooperation = 0,$club_type = '') {
       
        $this->show_club($keywords, $partnership_type, $project_id, $no_cooperation,$club_type,'club');
    }

    public function actionClubmore($keywords = '', $partnership_type = 0, $project_id = 0, $no_cooperation = 0,$club_type = '') {
      
     $this->show_club($keywords, $partnership_type, $project_id, $no_cooperation,$club_type,'clubmore');
    }

    function show_club($keywords, $partnership_type, $project_id, $no_cooperation,$club_type,$pfile ) {
        $data = array();
        $model = ClubList::model();
        $criteria = new CDbCriteria;
        //$criteria->condition = 'if_del=510';
        $criteria->select = 'id select_id,club_name select_title';
        if ($keywords != '') {
            $criteria->condition =get_like($criteria->condition,'club_name',$keywords,'');
        }
    
        parent::_list($model, $criteria, $pfile, $data);
    }

```



### 一：在ClubNews页面上取出PayType对应的数据库(pay_type)里的缴费类型进行遍历填写并结合ClubNews的项目类型一起传到ClubPaySet对应的数据库(hash)

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

models/ClubNews.php

注意：此处若有多个文件/图片上传则用逗号分隔

```php
    public function picLabels() {
        return 'imagesurl,costdetail';
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

### 需求十五：权限管理功能

### 需求十六：接口编写

微信小程序与php后台的接口

1.后台把数据交给微信小程序

wxOtherController.php

```php
<?php

class WxOtherController extends IoBaseController
{
    public $model;
    public $s;
    
    //导师将要进行的活动
    //http://localhost/sanli/index.php?r=WxOther/GetTeaAct
    public function actionGetTeaAct()
    {
        $para=getParameter('id');
        $s1='id,username,coursename,courseid';
        $s2='imagesurl,signIn_date_start:startTime,signIn_date_end:endTime';
        $w1="userid='".$para['id']."' and status=3";
        $data = teaSignList::model()->findAll($w1);
        $data1=array();
        $data2=array();
        $i=0;
        $today=date("Y-m-d H-i-s");

        foreach ($data as $v)
        {
            $courseid = $v->courseid;
            $tmp = ClubNews::model()->find('id ='.$courseid); //找到对应的活动
            if($tmp->signIn_date_start > $today)//活动尚未开始
            {
                $data1[$i] = $data[$i];
                $data2[$i] = $tmp;
                $i++;
            }
        }
        //put_msg($data1);
        $da1=toIoArray($data1,$s1);//在对象数组中取出$s1需要的变量变成数组传给$da1
        $da2=toIoArray($data2,$s2);
        $rs=array('data1'=>$da1,'data2'=>$da2);
        Basefun::model()->echoEncode($rs);
    }
}
```

在浏览器进行测试后把json信息导入文档中解析

![](https://s4.ax1x.com/2022/01/26/7qijzT.jpg)

### 需求十七：微信支付接口

小程序对接第三方支付是一件比较麻烦的事情，但是在实际项目中还是有很多商家想对接第三方支付，原因有很多：

1. 第一个是因为某些特殊需求，比如这个小程序有多个商家，比如也想在支付宝小程序进行支付等等；
2. 第二个某些领域是对接第三方支付的费率会比直接使用微信支付费率第一点；
3. 第三点是第三方支付有些可以打款至法人私户，微信支付好像只能打款至公户？（其实我也不太清楚）

反正因为各种各样的原因，甲方对第三方支付的需求很大，所以小程序对接第三方支付是较为常见的需求，但翻遍全网对第三方支付的对接资料较少，找了半天也没有多少资料可以参考。。。。写下这个来记录一波。

**支付流程**

1. 甲方向第三方支付机构提交进件，第三方机构代为申请微信支付

2. 第三方机构下方帐号和密钥，对应的第三方支付接口

   （第三方支付接口作用：提交下游订单号、支付金额、帐号、签名、商品描述、回调地址等信息后，第三方接口可返回第三方支付订单号、唤起微信支付的pay_info等信息）

3. 后台和小程序进行相应的开发

4. 开发完成后

5. 用户点击支付按钮

6. 小程序发送对应订单信息和商品信息至后台，后台根据签名规则生成对应的签名

7. 将接口需要的信息（包括签名）发送至第三方接口，第三方接口返回信息至后端

8. 后端发送第三方接口返回的信息至小程序

   （上述后端内容也可以在小程序写代码完成）

9. 小程序接收信息后，根据对应信息唤起微信支付

10. 微信支付唤起后，用户付款

11. 第一，小程序微信支付会返回支付是否成功信息

    第二，回调接口会返回支付是否成功信息

12. 前后端进行校验，前端检查微信支付返回的是否成功信息，如果成功，对后端发送SUCCESS，后端可使用回调接口返回信息与前端SUCCESS进行前后端校验，也可以后端为保险起见使用第三方支付给的查询订单接口查询并返回信息进行前后端校验。

13. 校验成功：将订单状态进行更改

    校验失败：返回错误信息，或调试程序。



支付接口文档 http://tgyapi.yltg.com.cn/project/35/interface/api/41

下图的文档中有需要获取的支付信息

![](https://s4.ax1x.com/2022/02/24/bkMp9A.jpg)

文档要求

![](https://s4.ax1x.com/2022/02/25/bkQ1xI.jpg)



后端controller

```php
	//支付接口
    public function actiontgPay()
    {
        $p = new tgPay();
        $request=getParameter("payMoney,lowOrderId,body,isMinipg");
        $request['account'] = 'XXXXXXXX'; //第三方下方的账号(接口要求)
        $request['appId'] 'XXXXXXXXX';	//小程序appId(接口要求)
        $res = $p->pay($request);
        $res = json_decode($res);
        $res -> request = $request;
        echo json_encode($res);	//第三方接口返回的数据送至小程序
    }

    //支付回调接口
    //传送参数是RW格式
    public function actionAccepttgPay()
    {
        $request = file_get_contents("php://input");
        $request = json_decode($request, true);
        $p = new tgCallback();
        $res = $p->call($request);
        echo $res;
    }

    //通莞支付查询上游订单接口并作校验 根据第三方接口需求做出调整
    public function actionorderFind($lowOrderId,$status)
    {
        $p = new orderFind();
        $res = $p->orderQuery($lowOrderId);
        $res = json_decode($res);
        $sign_order = SignList::model()->find("id=".$lowOrderId);
        if($status==="SUCCESS" && $res->state=="0")
        {
            if(!empty($sign_order))
            {
                $sign_order->status = 3;    //更改状态
                $sign_order->payid = $res->upOrderId;   //通莞支付id
                $sign_order->paychannel = "缴费通";    //支付方式
                $sign_order->payMoney = $res->payMoney; //缴费金 额
                $sign_order->payState = "支付成功"; //支付状态
                $sign_order->wxpayid = $res->channelOrderId;    //微信支付id
                $sign_order->save();
                $this->JsonSuccess(array('code'=>100,'msg'=>'支付成功'));
            }
            else
                $this->JsonSuccess(array('code'=>101,'msg'=>'数据库订单信息出错'));
        }
        else
        {
            $sign_order->status = 1; $sign_order->save();
            if($res->status==101) $this->JsonSuccess(array('code'=>101,'msg'=>$res->message));
            else{
                switch($res->state)
                {
                    case 1:
                        $this->JsonSuccess(array('code'=>101,'msg'=>'支付失败'));
                    case 2:
                        $this->JsonSuccess(array('code'=>101,'msg'=>'用户撤销支付'));
                    case 4:
                        $this->JsonSuccess(array('code'=>101,'msg'=>'待支付'));
                    case 5:
                        $this->JsonSuccess(array('code'=>101,'msg'=>'已转入退款'));
                    case 6:
                        $this->JsonSuccess(array('code'=>101,'msg'=>'已转入部分退款'));
                    default:
                        $this->JsonSuccess(array('code'=>101,'msg'=>'已转入部分退款'));
                }
            }

        }
    }
```

models/tgPay.php

```php
<?php
date_default_timezone_set('PRC');
class tgPay{

    public $url = 'https://ipay.833006.net/tgPosp/services/payApi/wxJspay';
    public $account = '25863396782801';
    public $privateKey = '456de823efab4485d4be220fdc7fbfb4';

    public function pay($request = array(
        'payMoney',
        'lowOrderId',
        'body',
        'isMinipg' => "1",
        'notifyUrl',
        'returnUrl',
        'openId',
        'appId',
        'attach',
        'storeid'
    )){
        $data = array(
            'account' => isset($request['account'])?$request['account']:$this->account,
            'payMoney' => $request['payMoney'], //单位元
            'lowOrderId' => $request['lowOrderId'], // 下游订单号
            'body' => $request['body'], // 商品描述
            'isMinipg' => $request['isMinipg'], // 是否小程序:1-是
            'notifyUrl' => $request['notifyUrl'],// 回调通知地址
            //'returnUrl' => $request['returnUrl'], // 交易完成后跳转的URl
            'openId' => $request['openId'], // 微信用户在公众号的唯一标识
            'appId' => $request['appId'], // 公众号 AppID
            //'attach' => $request['attach'], // 附加信息,
            //'storeid' => $request['storeid'], // 门店号
        );

        $data['sign'] = $this->getSign($data);//调用签名算法
        $result = $this->postByCurl($this->url, json_encode($data));

        str_replace('\"', '"', $result);
        return $result;
    }

    /**
     * 接口请求
     * @param $reqUrl
     * @param $json
     * @return mixed
     */
    function postByCurl($reqUrl, $json){
        $SSL = substr($reqUrl, 0, 8) == "https://"?true:false;
        $ch = curl_init(); // 初始化,创建一个新cURL资源

        if ($SSL) {
            curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false); // 信任任何证书
            curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 2); // 检查证书中是否设置域名
        }
        curl_setopt($ch, CURLOPT_HTTPHEADER, array(
                'Content-Type: application/json;charset=utf-8')
        );
        curl_setopt($ch, CURLOPT_URL, $reqUrl); // 要访问的地址
        curl_setopt($ch, CURLOPT_POSTFIELDS, $json);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true); // 执行结果是否被返回，0是返回，1是不返回
        $data = curl_exec($ch); // 执行并获取数据(抓取URL并把它传递给浏览器)初始化一个curl并且全部的选项都设置之后再调用。
//        $info = curl_getinfo($ch);
        curl_close($ch);  // 关闭cURL资源，并且释放系统资源
        return $data;
    }

    /**
     * 获取签名
     */
    public function getSign($condition)
    {
        $res = $this->ASCII($condition);//按ASCII码排序
        $res = $res.'&key='.$this->privateKey;//把密钥拼起来
        $res = strtoupper(md5($res));//转MD5码 并转字符串
        return  $res;
    }

    /**
     * ASCII排序
     */
    function ASCII($params = array()){
        if(!empty($params)){
            $p =  ksort($params);
            if($p){
                $str = '';
                foreach ($params as $k=>$val){
                    if($val != ''){
                        $str .= $k .'=' . $val . '&';
                    }
                }
                $strs = rtrim($str, '&';
                return $strs;
            }
        }
        return false;
    }
}


?>

```

models/tgCallback（回调）

```php
<?php
date_default_timezone_set('PRC');
class tgCallback{

    public $account = 'XXXXXXX'; //第三方下放账号
    public $privateKey = 'XXXXXXXX';	//第三方下放密钥

    public function call($data = array(
        "sign",
        "payMoney",
        "channelId",
        "orderDesc",
        "attach",
        "state",
        "upOrderId",
        "account",
        "openid",
        "merchantId",
        "settlementChannel",
        "payTime",
        "payoffType",
        "lowOrderId",
    )){
        // 验证签名
        $sign = $data['sign'];
        unset($data['sign']);
        $check_sign = $this->getSign($data);
        if($check_sign != $sign){
            return '验签失败';
        }

        return 'SUCCESS';	//第三方支付接口有要求返回SUCCESS
    }

    /**
     * 获取签名
     */
    public function getSign($condition)
    {
        $res = $this->ASCII($condition);
        $res = $res.'&key='.$this->privateKey;
        $res = strtoupper(md5($res));
        return  $res;
    }

    /**
     * ASCII排序
     */
    function ASCII($params = array()){
        if(!empty($params)){
            $p =  ksort($params);
            if($p){
                $str = '';
                foreach ($params as $k=>$val){
                    if($val != ''){
                        $str .= $k .'=' . $val . '&';
                    }
                }
                $strs = rtrim($str, '&');
                return $strs;
            }
        }
        return false;
    }
}

?>

```

models/reverse.php

```php
<?php
date_default_timezone_set('PRC');
class reverse{
    
    public $account = '25863396782801';
    public $privateKey = '456de823efab4485d4be220fdc7fbfb4';

    public function call($data = array(
        "sign",
        "payMoney",
        "channelId",
        "orderDesc",
        "attach",
        "state",
        "upOrderId<?php
date_default_timezone_set('PRC');
class tgPay{

    public $url = 'XXXXXXXXX'; //第三方支付接口地址
    public $account = 'XXXXXXXXX';	//第三方支付下发账号
    public $privateKey = 'XXXXXXXXXXXX';	//第三方支付下发密钥

    public function pay($request = array(
        'payMoney',
        'lowOrderId',
        'body',
        'isMinipg' => "1",
        'notifyUrl',
        'openId',
        'appId'
    )){
        $data = array(
            'account' => isset($request['account'])?$request['account']:$this->account,
            'payMoney' => $request['payMoney'], //单位元
            'lowOrderId' => $request['lowOrderId'], // 下游订单号
            'body' => $request['body'], // 商品描述
            'isMinipg' => $request['isMinipg'], // 是否小程序:1-是
            'notifyUrl' => "http://XXXXXXXXX",// 回调通知地址，最好不要使用https，有可能无法接受回调信息。
            //'returnUrl' => $request['returnUrl'], // 交易完成后跳转的URl
            'openId' => $request['openId'], // 微信用户在公众号的唯一标识
            'appId' => $request['appId'], // 公众号 AppID
            //'attach' => $request['attach'], // 附加信息,
            //'storeid' => $request['storeid'], // 门店号
        );
        $data['sign'] = $this->getSign($data);	//制作签名
        $result = $this->postByCurl($this->url, json_encode($data));	//发送请求至第三方接口，返回result

        str_replace('\"', '"', $result);
        return $result;
    }

    /**
     * 接口请求
     * @param $reqUrl
     * @param $json
     * @return mixed
     */
    function postByCurl($reqUrl, $json){
        $SSL = substr($reqUrl, 0, 8) == "https://"?true:false;
        $ch = curl_init(); // 初始化,创建一个新cURL资源

        if ($SSL) {
            curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false); // 信任任何证书
            curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 2); // 检查证书中是否设置域名
        }
        curl_setopt($ch, CURLOPT_HTTPHEADER, array(
                'Content-Type: application/json;charset=utf-8')
        );
        curl_setopt($ch, CURLOPT_URL, $reqUrl); // 要访问的地址
        curl_setopt($ch, CURLOPT_POSTFIELDS, $json);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true); // 执行结果是否被返回，0是返回，1是不返回
        $data = curl_exec($ch); // 执行并获取数据(抓取URL并把它传递给浏览器)初始化一个curl并且全部的选项都设置之后再调用。
//        $info = curl_getinfo($ch);
        curl_close($ch);  // 关闭cURL资源，并且释放系统资源
        return $data;
    }

    /**
     * 获取签名
     */
    public function getSign($condition)
    {
        $res = $this->ASCII($condition);
        $res = $res.'&key='.$this->privateKey;
        $res = strtoupper(md5($res));
        return  $res;
    }

    /**
     * ASCII排序
     */
    function ASCII($params = array()){
        if(!empty($params)){
            $p =  ksort($params);
            if($p){
                $str = '';
                foreach ($params as $k=>$val){
                    if($val != ''){
                        $str .= $k .'=' . $val . '&';
                    }
                }
                $strs = rtrim($str, '&');
                return $strs;
            }
        }
        return false;
    }
}


?>
        "account",
        "openid",
        "merchantId",
        "settlementChannel",
        "payTime",
        "payoffType",
        "lowOrderId",
    )){
        // 验证签名
        $sign = $data['sign'];
        unset($data['sign']);

        $check_sign = $this->getSign($data);

        if($check_sign != $sign){
            return '验签失败';
        }

        return json_encode($data);
    }

    /**
     * 获取签名
     */
    public function getSign($condition)
    {
        $res = $this->ASCII($condition);
        $res = $res.'&key='.$this->privateKey;
        $res = strtoupper(md5($res));
        return  $res;
    }

    /**
     * ASCII排序
     */
    function ASCII($params = array()){
        if(!empty($params)){
            $p =  ksort($params);
            if($p){
                $str = '';
                foreach ($params as $k=>$val){
                    if($val != ''){
                        $str .= $k .'=' . $val . '&';
                    }
                }
                $strs = rtrim($str, '&');
                return $strs;
            }
        }
        return false;
    }
}

?>


```

orderFind(订单查找，用于校验，为什么不用回调信息校验，因为我觉得回调给的信息太少)

```php
<?php
date_default_timezone_set('PRC');
class orderFind{
    
    public $url = 'https://XXXXXXXX';//查询订单接口
    public $account = 'XXXXXXX'; //第三方下放账号
    public $privateKey = 'XXXXXXXX';	//第三方下放密钥

    public function orderQuery($lowOrderId){
        $data = array(
            'account' => $this->account,
            'lowOrderId' => $lowOrderId // 下游订单号
        );

        $data['sign'] = $this->getSign($data);
        $result = $this->postByCurl($this->url, json_encode($data));

        str_replace('\"', '"', $result);
        return $result;
    }

    /**
     * 接口请求
     * @param $reqUrl
     * @param $json
     * @return mixed
     */
    function postByCurl($reqUrl, $json){
        $SSL = substr($reqUrl, 0, 8) == "https://"?true:false;
        $ch = curl_init(); // 初始化,创建一个新cURL资源

        if ($SSL) {
            curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false); // 信任任何证书
            curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 2); // 检查证书中是否设置域名
        }
        curl_setopt($ch, CURLOPT_HTTPHEADER, array(
                'Content-Type: application/json;charset=utf-8')
        );
        curl_setopt($ch, CURLOPT_URL, $reqUrl); // 要访问的地址
        curl_setopt($ch, CURLOPT_POSTFIELDS, $json);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true); // 执行结果是否被返回，0是返回，1是不返回
        $data = curl_exec($ch); // 执行并获取数据(抓取URL并把它传递给浏览器)初始化一个curl并且全部的选项都设置之后再调用。
//        $info = curl_getinfo($ch);
        curl_close($ch);  // 关闭cURL资源，并且释放系统资源
        return $data;
    }

    /**
     * 获取签名
     */
    public function getSign($condition)
    {
        $res = $this->ASCII($condition);
        $res = $res.'&key='.$this->privateKey;
        $res = strtoupper(md5($res));
        return  $res;
    }

    /**
     * ASCII排序
     */
    function ASCII($params = array()){
        if(!empty($params)){
            $p =  ksort($params);
            if($p){
                $str = '';
                foreach ($params as $k=>$val){
                    if($val != ''){
                        $str .= $k .'=' . $val . '&';
                    }
                }
                $strs = rtrim($str, '&');
                return $strs;
            }
        }
        return false;
    }
}
```

微信小程序 pay/confiem.js

```javascript
checkSubmit: function(e) { //支付按钮
        var that = this;
        wx.request({
            url: app.globalData.url + "WxSign/tgPay", //后端支付接口
            data:{	//数据根据第三方支付接口需求进行发送
                lowOrderId:that.options.orderid,  //后台雪花算法生成的订单号
                payMoney:0.01, //支付金额
                body:that.data.title,   //商品描述
                //notifyUrl:app.globalData.url + "WxSign/AccepttgPay",  //回调地址
                isMinipg:1, //是否是小程序
                openId:that.data.userinfo.openid,   //openId
            },
            method: 'GET', // OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT
            // header: {}, // 设置请求的 header
            success: function(res) {
                var pay_res = JSON.parse(res.data.pay_info); //由于第三方接口返回原生json字符串，需解码后才能进行使用
                wx.requestPayment({   //唤起微信支付
                    timeStamp: pay_res.timeStamp.toString(),
                    nonceStr: pay_res.nonceStr,
                    package: pay_res.package,
                    signType: pay_res.signType,
                    paySign: pay_res.paySign,
                    success(res) {
                        wx.request({
                            url: app.globalData.url + "WxSign/orderFind",	//进行校验
                            data:{
                                lowOrderId:that.options.orderid,  //后台雪花算法生成的订单号
                                status:"SUCCESS",	//前端发送SUCCESS标志
                            },
                            method:'GET',
                            success:function(res){
                                console.log(res);
                                wx.showToast({
                                    title: '购买成功',
                                    icon: 'success'
                                })
                            }                        
                        })
                    },
                    fail(res) {
                        console.log(res);
                        wx.showToast({
                            title: '购买失败',
                            icon: 'none'
                        })
                    }
                  })
                  
                }
            })
        }
    },

    // 获取设备信息
    getSysInfo: function(e) {
        wx.getSystemInfo({
            success: (result) => {
                this.setData({
                    screenHeight: (result.screenHeight - result.statusBarHeight)
                })
            },
        })
    },
```

### 需求十八： 导师学生定位功能

功能：

导师可请求获得学生的位置（学生上次上次的位置），学生端每隔30秒检查一次，导师是否请求位置，若有请求则更新位置

微信app.js

```javascript
//app.js
import config from 'config.js';
App({
    onLaunch: function() {
        var that = this;
        // Do something initial when launch.
        //调用登录接口
        "pages/index/index", {
            "pagePath": "pages/register_stu/register_stu",
            "text": "注册",
            "iconPath": "img/shop.png",
            "selectedIconPath": "img/shop2.png"
        }
        // return;

        let menuButtonObject = wx.getMenuButtonBoundingClientRect();
        //this.getSystemInfo();
        //  return ;
        wx.getSystemInfo({
                success: (result) => {
                    this.globalData.statusHeight = result.statusBarHeight;
                    this.globalData.height = result.screenHeight;
                    this.globalData.canHeight = result.windowHeight;
                    this.globalData.width = result.screenWidth;
                },
            })
    },

    //获取用户信息
    getUserInfo: function() {
        var that = this;
        var user_info = wx.getStorageSync('user_info');
        if (!user_info) {
            that.show_page('/pages/login/login');
        } else
            that.globalData.userInfo = user_info;
        //  return user_info;
    },
    //获取设备信息
    getSystemInfo: function() {
        let menuButtonObject = wx.getMenuButtonBoundingClientRect();
        if (this.globalData.systemInfo) {
            return this.globalData.systemInfo;
        } else {
            wx.getSystemInfo({
                success: (res) => {

                    this.globalData.systemInfo = res;
                    let statusBarHeight = res.statusBarHeight,
                        navTop = menuButtonObject.top, //胶囊按钮与顶部的距离
                        navHeight = statusBarHeight + menuButtonObject.height + (menuButtonObject.top - statusBarHeight) * 2; //导航高度
                    this.globalData.navHeight = navHeight;
                    this.globalData.navTop = navTop;
                    this.globalData.windowHeight = res.windowHeight;
                    return this.globalData.systemInfo;
                    // typeof cb == "function" && cb(_this.globalData.systemInfo);
                }
            })
        }
    },


    set_address: function(paddress) {
        this.globalData.indexProvince = paddress.indexProvince;
        this.globalData.indexCity = paddress.indexCity;
        this.globalData.indexDistrict = paddress.indexDistrict;
        this.globalData.province = paddress.province;
        this.globalData.dity = paddress.dity;
        this.globalData.district = paddress.district;
        this.globalData.detailedInfo = paddress.detailedInfo;
    },

    globalData: {
        userInfo: null,
        indexProvince: 0,
        indexCity: 0,
        indexDistrict: 0,
        province: '',
        dity: '',
        district: '',
        detailedInfo: '',
        userOpenId: 'undefined',
        addressList: [],
        userId: 0,
        showp: 0,
        login: 141, //登录的用户的id
        loginr1: 0,
        loginr2: 0,
        viewMore: '',
        otherAddressInfo: null,
        isCompleteInfo: 0, //是否完成报名a
        isTimeEnd: "0", //计时结束
        url: config.url, //控制器路径
        openid: "",
        // 状态栏高度
        statusHeight: "",
        // 设备高度
        height: "",
        // 可使用高度
        canHeight: "",
        // 设备宽度
        width: "",
        //记录账号是学生还是教师还是家长，默认是学生
        flag_identity: [1, 0, 0],
        navigate_name: "",
        //经纬度
        latitude: "",
        longtitude: "",
        realTime: null, //实时数据对象(用于关闭实时刷新方法)
        // imgUrl: 'https://shenhailao.com/hsreport/uploads/temp/WxImg/',
        imgUrl: 'https://sanli-tracks.com/sanli/uploads/temp/WxImg/',
        // imgUrl: 'http://localhost/gitSanli/sanli/uploads/temp/WxImg/',
        // okayapiHost: "http://test_phalapi.com", // TODO: 配置成你所在的接口域名
        okayApiAppKey: "appkey", // TODO：改为你的APP_KEY 在http://open.yesapi.cn/?r=App/Mine寻找
        okayApiAppSecrect: "appsecret" // TODO：改为你的APP_SECRECT
    },

    show_msg: function(msg) {
        wx.showToast({ title: msg, duration: 2000, });
    },
    show_error: function() {
        $this.show_msg('网络异常！');
    },

    show_page: function(myurl) {
        wx.navigateTo({
            url: myurl,
            success: function(res) {},
            fail: function(res) {
                if (res.errMsg && (res.errMsg == 'navigateTo:fail can not navigateTo a tabbar page' || res.errMsg == 'navigateTo:fail can not navigate to a tabbar page')) {

                    wx.switchTab({ url: myurl, })
                }
            },
            complete: function(res) {
                // console.log('res', res)
            },
        })
    },

    checkUserInfo: function() {
        var that = this;
        var s1 = "";
        s1 = s1 + '&nickName=' + this.globalData.userInfo.nickName;
        s1 = s1 + '&avatarUrl=' + this.globalData.userInfo.avatarUrl;
        wx.request({
            url: this.globalData.url + 'CheckUser' + s1,
            method: 'POST',
            data: { formData: 0 }, //,formId:formId,openId:app.globalData.openId
            header: {
                'Content-Type': 'application/json'
            },
            success: function(res) {
                var responseData = res.data.code;
                s1 = '/pages/ulogin/login';
                if (responseData) { //意见备案，
                    that.globalData.userId = res.data.userId;
                    s1 = '/pages/index/index';
                }
                that.show_page(s1);
            },
            fail: function(res) {
                that.show_page('/pages/ulogin/login');
            },
        });
    },
    //导师获取正在活动的数据
    //获取学生信息
    TeaNowCourse: function() {
        var that = this
        let user = wx.getStorageSync('user');

        var id = "";
        wx.request({
            url: that.globalData.url + 'WxSign/TeaNowCourse&id=' + user.id,//取导师正在进行的活动
            data: {},
            method: 'GET', // OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT
            // header: {}, // 设置请求的 header
            success: function(res) {
                // success
                console.log("导师获取正在活动数据")
                console.log(res)
                console.log(res.data.data[0])
                if (res.data.data[0] != "无正在进行的课程") {
                    id = res.data.data[0].courseid;
                    that.GetLocation(id); //获取老师的位置
                } 

            },
            fail: function() {
                // fail
            },
            complete: function() {
                // complete
            }
        })
    },
    //学生获取正在活动的数据
    ActiveStuDetail: function() {
        let user = wx.getStorageSync('user')
        var that = this
        var id = "",
            courseid = 1;
        wx.request({
            url: that.globalData.url + 'WxSign/ActiveStuDetail',
            data: {
                id: user.id,
            },
            method: 'GET', // OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT
            // header: {}, // 设置请求的 header
            success: function(res) {
                // success
                console.log("学生获取正在活动数据");
                console.log(res)
                console.log(res.data.data[0])
                if (res.data.data[0] == "无正在进行的课程") {

                } else {
                    //setTimeout是一个定时器，定时到期后执行回调函数
                    that.globalData.realTime = setTimeout(function() {
                        courseid = res.data.data[0].courseid;
                        wx.request({
                            url: that.globalData.url + 'WxOther/GpsAccept',//学生接收导师获取GPS请求
                            data: {
                                courseid: res.data.data[0].courseid,
                                teacherid: res.data.data[0].teacherid,
                            },
                            method: 'GET', // OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT
                            // header: {}, // 设置请求的 header
                            success: function(res) {
                                // success
                                console.log("学生接收的请求是")
                                console.log(res)
                                if (res.data.data.code == 1) {
                                    that.GetLocation(courseid); //获取学生的位置
                                }
                            },
                            fail: function() {
                                // fail
                            },
                            complete: function() {
                                // complete
                            }
                        })
                    }, 30000) //每隔30秒查看一次导师是否发送请求获取学生的GPS
                }
            },
            fail: function() {
                // fail
            },
            complete: function() {
                // complete
            }
        })
    },
    onShow: function() {
        let id_flag = wx.getStorageSync("id_flag");
        let user = wx.getStorageSync('user')
        console.log(id_flag)
        console.log(user);
        var that = this;
        if (user != null && user != '') {
            // that.globalData.realTime = setInterval(function() {
            // 请求服务器数据

            if (id_flag == "teacher") {
                that.TeaNowCourse()

            }
            if (id_flag == 'student') {
                that.ActiveStuDetail();
            }

            // }, 30000) //间隔时间

            // 更新数据
            that.globalData.realTime = that.globalData.realTime; //实时数据对象(用于关闭实时刷新方法)

        }
    },
    //上传学生定位
    UpStuLocation: function(courseid, latitude, longitude) {
        let user = wx.getStorageSync('user');
        let that = this;
        wx.request({
            url: that.globalData.url + 'WxOther/UpGps',
            data: {
                userid: user.id,
                courseid: courseid,
                longitude: longitude,
                latitude: latitude,
            },
            method: 'GET', // OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT
            // header: {}, // 设置请求的 header
            success: function(res) {
                // success
                console.log("成功上传学生位置");
                console.log(res)
            },
            fail: function() {
                // fail
            },
            complete: function() {
                // complete
            }
        })
    },
    //上传导师定位
    UpTeaLocation: function(id, latitude, longitude) {
        let id_flag = wx.getStorageSync("id_flag");
        let user = wx.getStorageSync('user')
        wx.request({
            url: this.globalData.url + 'WxOther/TeaUpGps',
            data: {
                userid: user.id,
                courseid: id,
                longitude: longitude,
                latitude: latitude,
            },
            method: 'GET', // OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT
            // header: {}, // 设置请求的 header
            success: function(res) {
                // success
                console.log("成功上传位置信息")
                console.log(res)
            },
            fail: function() {
                // fail
            },
            complete: function() {
                // complete
            }
        })
    },
    // 获取定位(老师或学生)
    GetLocation: function(id) {

        let id_flag = wx.getStorageSync("id_flag");//取出缓存数据判断当前用户是老师还是学生
        let user = wx.getStorageSync('user')
        wx.getLocation({
            type: "gcj02",
            altitude: 'true',
            isHighAccuracy: 'true',
            highAccuracyExpireTime: '3500',
            success: (res) => {
                this.globalData.latitude = res.latitude;
                this.globalData.longtitude = res.longitude;
                console.log("导师或学生上传位置")
                console.log(res.latitude);
                console.log(res.longitude)
                if (id_flag == 'teacher') { this.UpTeaLocation(id, this.globalData.latitude, this.globalData.longtitude) }
                if (id_flag == 'student') { this.UpStuLocation(id, this.globalData.latitude, this.globalData.longtitude) }

            },
            fail: (res) => {
                wx.showToast({
                    title: '获取失败',
                    icon: 'error',
                    duration: 800
                })
                console.log(res);
            }
        })
    },

    /**
     * 生命周期函数--监听页面隐藏
     */
    onHide: function() {
        /**
         * 当页面隐藏时关闭定时器(关闭实时刷新)
         * 切换到其他页面了
         */
        clearInterval(this.globalData.realTime)
    },
})

```

models

Gps.php

```php
<?php

class Gps extends BaseModel {


    public $location ='';
    public function tableName() {
        return '{{gps_request}}';
    }

    /**
     * 模型验证规则
     */
    public function rules() {
        return array(
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
            'id'=>'ID',
            'time'=>'时间戳',
            'courseid'=>'课程id',
            'course'=>'课程名字',
            'teacherid'=>'老师id',
            'teacher'=>'老师名字',

        );
    }

    /**
     * Returns the static model of the specified AR class.
     */
    public static function model($className = __CLASS__) {
        return parent::model($className);
    }

    protected function beforeSave() {
        parent::beforeSave();
        return true;
    }

    protected function afterFind()
    {
        return parent::afterFind();
    }
}

```

后端controller

```php
    //获取导师正在进行的活动
    //传入参数：教师id
    //获取参数：学生报名和分组信息，导师信息
    //http://localhost/sanli/index.php?r=WxSign/TeaNowCourse
    public function actionTeaNowCourse($id)
    {
        $s1 = 'userid,username,photo,sex,groupnum,room,car,hotel,coursename,longitude,latitude,GPS_updatetime,courseid';
        $criteria=new CDbCriteria();
        $time=date("Y-m-d H:i:s");
        $criteria ->condition=get_where('1=1',$id,'teacherid',$id,'"');
        $criteria ->condition=get_where($criteria ->condition,3,'status',3,'');
        $criteria ->condition=get_where($criteria ->condition,$time,'starttime<=',$time,'"');
        $criteria ->condition=get_where($criteria ->condition,$time,'endtime>=',$time,'"');
        $da1 =SignList::model()->recToArray($criteria,$s1);
        if(empty($da1)){
            $this->JsonFail(array('无正在进行的课程'));
        }
        else{
            $rs=array('data'=>$da1);
            Basefun::model()->echoEncode($rs);
        }
    }


    //获取正在的活动列表接口(家长、学生)
    //传入参数：学生id
    //获取参数：学生报名和分组信息，导师信息
    //http://localhost/sanli/index.php?r=WxSign/ActiveStuDetail
    public function actionActiveStuDetail($id)
    {
        $s1 = 'userid,username,sex,grade,class,courseid,coursename,teacherid,photo,';
        $s1.= 'room,car,groupnum,hotel,t_phone,starttime,endtime';
        $criteria=new CDbCriteria();
        $time=date("Y-m-d H:i:s");
        $criteria ->condition=get_where('1=1',$id,'userid',$id,'"');
        $criteria ->condition=get_where($criteria ->condition,3,'status',3,'"');
        $criteria ->condition=get_where($criteria ->condition,$time,'starttime<=',$time,'"');
        $criteria ->condition=get_where($criteria ->condition,$time,'endtime>=',$time,'"');
        $da1 =SignList::model()->recToArray($criteria,$s1);
        if(empty($da1)){
            $this->JsonFail(array('无正在进行的课程'));
        }
        else{
            $teacherid = $da1[0]['teacherid'];
            $s2 = 'name,header,sex,phone';
            $da2 =Teacher::model()->recToArray("id=".$teacherid,$s2);
            $rs=array('data'=>$da1,'teacher'=>$da2);
            Basefun::model()->echoEncode($rs);
        }
    }


    //学生接收导师获取GPS请求
    //http://localhost/sanli/index.php?r=WxOther/GpsAccept
    //传参：courseid teacherid
    //获得参数：是否上传GPS：1/0
    public function actionGpsAccept($courseid,$teacherid){
        $criteria=new CDbCriteria();
        $criteria ->condition=get_where("1=1",$courseid,'courseid',$courseid,'"');
        $criteria ->condition=get_where($criteria ->condition,$teacherid,'teacherid',$teacherid,'"');
        $da1 =Gps::model()->find($criteria);
        if(empty($da1)) {
            $this->JsonSuccess(array('code'=>0,'msg'=>'无获取请求'));
        }
        else if(strtotime('now')-$da1->time<=35 || strtotime('now')-$da1->time>10*60){
            $this->JsonSuccess(array('code'=>1,'msg'=>'导师正在获取GPS'));
        }//strtotime将文本日期时间转换为Unix时间戳
        else{
            $this->JsonSuccess(array('code'=>0,'msg'=>'无获取请求'));
        }
    }


    //学生上传GPS信息
    //http://localhost/sanli/index.php?r=WxOther/UpGps
    //传参：userid courseid latitude longitude
    //获得参数：是否成功上传
    public function actionUpGps($userid,$courseid,$longitude,$latitude){
        $criteria=new CDbCriteria();
        $criteria ->condition=get_where("1=1",$courseid,'courseid',$courseid,'"');
        $criteria ->condition=get_where($criteria ->condition,$userid,'userid',$userid,'"');
        $da1 =SignList::model()->find($criteria);
        if(empty($da1)) {
            $this->JsonFail(array('msg'=>'请求错误'));
        }
        else{
            $da1->longitude = $longitude;
            $da1->latitude = $latitude;
            $da1->GPS_updatetime = date("Y-m-d H:i:s");
            $flag = $da1->save();
        }
        $flag?$this->JsonSuccess(array('msg'=>'上传位置成功')):$this->JsonSuccess(array('msg'=>'上传位置失败'));
    }

    //导师上传GPS信息
    //http://localhost/sanli/index.php?r=WxOther/TeaUpGps
    //传参：userid courseid latitude longitude
    //获得参数：是否成功上传
    public function actionTeaUpGps($userid,$courseid,$longitude,$latitude){
        $criteria=new CDbCriteria();
        $criteria ->condition=get_where("1=1",$courseid,'courseid',$courseid,'"');
        $criteria ->condition=get_where($criteria ->condition,$userid,'userid',$userid,'"');
        $da1 =teaSignList::model()->find($criteria);
        //put_msg($longitude);
        $flag=false;
        if(empty($da1)) {
            //put_msg("获取失败");
            $this->JsonFail(array('msg'=>'请求错误'));
        }
        else{
            $da1->longitude = $longitude;
            $da1->latitude = $latitude;
            $da1->GPS_updatetime = date("Y-m-d H:i:s");
            /*put_msg("经纬度");
            put_msg($latitude);
            put_msg($longitude);*/
            $flag = $da1->save();
        }
        $flag?$this->JsonSuccess(array('msg'=>'上传位置成功')):$this->JsonSuccess(array('msg'=>'上传位置失败'));
    }
```

### 需求十九：学生上传信息实现自动分班统计

需求：学生每上传一次信息，自动更新班级信息，并统计班级数据

表：班级表  class   学生信息表:userinfo

models/Userinfo.php

从班级表里面查找，查不到就new一个新班级

```php
   protected function beforeSave() {
        parent::beforeSave();
        if($this->isNewRecord)
        {
            if(isset($_SESSION['adminid'])){    //自动保存上传人身份
                $this->update_userid=$_SESSION['adminid'];
                $this->update_username=$_SESSION['name'];
                $this->update_unitid=$_SESSION['TUNIT_id'];
                $this->update_unitname=$_SESSION['TUNIT'];
            }
            $time=date("Y-m-d");
            $this->registerdate=$time;
        }
        $school=$this->schoolname;
        $class=$this->class;
        $grade=$this->grade;
        $criteria = new CDbCriteria;
        $criteria->condition=get_where('1=1',$school,'school',$school,'"');
        $criteria->condition=get_where($criteria->condition,$class,'class',$class,'"');
        $criteria->condition=get_where($criteria->condition,$grade,'grade',$grade,'"');
        $tmp=BaseClass::model()->find($criteria);
        if(empty($tmp))
        {
            $tmp = new BaseClass();
            $tmp->school = $school;
            $tmp->grade = $grade;
            $tmp->class = $class;
            $tmp->province=$this->province;
            $tmp->city=$this->city;
            $tmp->district=$this->district;
            $tmp->save();
        }
        return true;
    }

```

在学生报名分班查看页面查看每个班报名和未完成报名情况



![](https://s1.ax1x.com/2022/03/15/bvR0bV.jpg)

ClassDisplay/index

```php
<div class="box">
    <?php echo show_title($this);?>
    <div class="box-content">
        <div class="box-header">
            <a class="btn" href="javascript:;" onclick="we.reload();"><i class="fa fa-refresh"></i>刷新</a>
            <a style="display:none;" id="j-delete" class="btn" href="javascript:;" onclick="we.dele(we.checkval('.check-item input:checked'), deleteUrl);"><i class="fa fa-trash-o"></i>刪除</a>
        </div><!--box-header end-->
        <div class="box-search">
            <form action="<?php echo Yii::app()->request->url;?>" method="get">
                <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">
                <input type="hidden" name="list_type" value="<?php echo Yii::app()->request->getParam('list_type');?>">
                <label style="margin-right:10px;">
                    <span>关键字：</span>
                    <input style="width:200px;" class="input-text" type="text" name="keywords" value="<?php echo Yii::app()->request->getParam('keywords');?>">
                </label>

                <button class="btn btn-blue" type="submit">查询</button>
            </form>
        </div><!--box-search end-->
        </form>
    </div><!--box-search end-->

    <div class="box-table">
        <table class="list">
            <thead>
            <tr>
                <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
                <th style='text-align: center;'>编号</th>
                <th style='text-align: center;'>学校</th>
                <th style='text-align: center;'>年级</th>
                <th style='text-align: center;'>班级</th>
                <th style='text-align: center;'>活动名称</th>
                <th style='text-align: center;'>报名中/报名完成人数</th>
                <th colspan="2" style='text-align: center;'>学生查看</th>
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
                    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
                    <td style='text-align: center;'><?php echo $v->school; ?></td>
                    <td style='text-align: center;'><?php echo $v->grade; ?></td>
                    <td style='text-align: center;'><?php echo $v->class; ?></td>
                    <td style='text-align: center;'><?php echo $v->coursename; ?></td>
                    <td style='text-align: center;'><?php echo $v->res_num; ?></td>
                    <td style='text-align: center;'>
                        <a class="btn btn-blue" href="<?php echo $this->createUrl('ClassDisplay/success', array('school'=>$v->school,'class'=>$v->class,'grade'=>$v->grade,'courseid'=>$v->courseid));?>" title="报名完成学生"><i class="fa fa-table "> 报名完成学生</i></a>
                    </td>
                    <td style='text-align: center;'>
                        <a class="btn btn-blue" href="<?php echo $this->createUrl('ClassDisplay/failed', array('school'=>$v->school,'class'=>$v->class,'grade'=>$v->grade,'courseid'=>$v->courseid));?>" title="未完成学生"><i class="fa fa-table "> 未完成学生</i></a>
                    </td>
                    <td><a class="btn" href="javascript:;" onclick="we.dele('<?php echo $v->id;?>', deleteUrl);" title="删除"><i class="fa fa-trash-o"></i></a></td>
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

ClassDisplay/faild.php

```php
<div class="box">
    <?php echo show_title($this,"未报名学生")?>
    <div class="box-content">
        <div class="box-header">
            <a class="btn" href="javascript:;" onclick="we.reload();"><i class="fa fa-refresh"></i>刷新</a>
        </div><!--box-header end-->
        <div>

        <form action="<?php echo Yii::app()->request->url;?>" method="get">
            <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">
            <input type="hidden" name="school" value="<?php echo Yii::app()->request->getParam('school');?>">
            <input type="hidden" name="class" value="<?php echo Yii::app()->request->getParam('class');?>">
            <input type="hidden" name="grade" value="<?php echo Yii::app()->request->getParam('grade');?>">
            <input type="hidden" name=courseid" value="<?php echo Yii::app()->request->getParam('courseid');?>">
                <label style="margin-right:10px;">
                    <span>根据姓名查询：</span>
                    <input style="width:200px;" class="input-text" type="text" name="keywords" value="<?php echo Yii::app()->request->getParam('keywords');?>">
                </label>
                <button class="btn btn-blue" type="submit" array=>查询</button>
        </form>
    </div><!--box-search end-->

        <div class="box-table">
            <table class="list">
                <thead>

                <tr>
                    <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
                    <th style='text-align: center;'>编号</th>
                    <th style='text-align: center;'>用户姓名</th>
                    <th style='text-align: center;'>活动名称</th>
                    <th style='text-align: center;'>状态</th>
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
                        <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
                        <td style='text-align: center;'><?php echo $v->name; ?></td>
                        <td style='text-align: center;'><?php echo $coursename; ?></td>
                        <td style='text-align: center;'><?php echo '<span style="color: red">'."未完成报名</span>"; ?></td>
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


```

ClassDisplay/success.php

```php
<div class="box">
    <?php echo show_title($this,"已报名学生")?>
    <div class="box-content">
        <div class="box-header">
            <a class="btn" href="javascript:;" onclick="we.reload();"><i class="fa fa-refresh"></i>刷新</a>
        </div><!--box-header end-->
        <div>

        <form action="<?php echo Yii::app()->request->url;?>" method="get">
            <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">
            <input type="hidden" name="school" value="<?php echo Yii::app()->request->getParam('school');?>">
            <input type="hidden" name="class" value="<?php echo Yii::app()->request->getParam('class');?>">
            <input type="hidden" name="grade" value="<?php echo Yii::app()->request->getParam('grade');?>">
            <input type="hidden" name=courseid" value="<?php echo Yii::app()->request->getParam('courseid');?>">
                <label style="margin-right:10px;">
                    <span>根据姓名查询：</span>
                    <input style="width:200px;" class="input-text" type="text" name="keywords" value="<?php echo Yii::app()->request->getParam('keywords');?>">
                </label>
                <button class="btn btn-blue" type="submit" array=>查询</button>
        </form>
    </div><!--box-search end-->

        <div class="box-table">
            <table class="list">
                <thead>

                <tr>
                    <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
                    <th style='text-align: center;'>编号</th>
                    <th style='text-align: center;'>用户姓名</th>
                    <th style='text-align: center;'>报名活动</th>
                    <th style='text-align: center;'>报名时间</th>
                    <th style='text-align: center;'>状态</th>
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
                        <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
                        <td style='text-align: center;'><?php echo $v->username; ?></td>
                        <td style='text-align: center;'><?php echo $v->coursename; ?></td>
                        <td style='text-align: center;'><?php echo $v->registrationtime; ?></td>
                        <td style='text-align: center;'><?php echo $v->status==1?'<span style="color: red">'."报名不成功</span>":($v->status==2?"报名成功未付款":($v->status==3?'<span style="color: lightseagreen">'."报名成功并付款</span>":"")); ?></td>
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


```

ClassDisplayController.php

```php
    
    public function actionIndex($keywords = '') {
        set_cookie('_currentUrl_', Yii::app()->request->url);
        $modelName = $this->model;
        $model = $modelName::model();
        $criteria = new CDbCriteria;
        $criteria->condition='1=1';
        $criteria->condition=get_like($criteria->condition,'school',$keywords,'');
        if($_SESSION['F_ROLENAME']==='学校')
            $criteria->condition=get_where($criteria->condition,$_SESSION['TUNIT'],'school',$_SESSION['TUNIT'],'"');
        if($_SESSION['F_ROLENAME']==='省级主管部门')
            $criteria->condition=get_where($criteria->condition,$_SESSION['F_province'],'province',$_SESSION['F_province'],'"');
        if($_SESSION['F_ROLENAME']==='市级主管部门')
            $criteria->condition=get_where($criteria->condition,$_SESSION['F_city'],'city',$_SESSION['F_city'],'"');
        if($_SESSION['F_ROLENAME']==='县级主管部门')
            $criteria->condition=get_where($criteria->condition,$_SESSION['F_district'],'district',$_SESSION['F_district'],'"');
            $data = array();
        parent::_list($model, $criteria, 'index', $data,20);
    }

	public function actionSuccess($school = '',$grade = '',$class = '',$courseid = '',$keywords = '') {
        set_cookie('_currentUrl_', Yii::app()->request->url);
        $model = SignList::model();
        $criteria = new CDbCriteria;
        $data = array();
        $criteria->condition=get_where('1=1',$courseid,'courseid',$courseid,'"');
        $criteria->condition=get_where($criteria->condition,$school,'school',$school,'"');
        $criteria->condition=get_where($criteria->condition,$grade,'grade',$grade,'"');
        $criteria->condition=get_where($criteria->condition,$class,'class',$class,'"');
        $criteria->condition=get_where($criteria->condition,3,'status',3,'"');
        $criteria->condition=get_like($criteria->condition,'username',$keywords,'');
        parent::_list($model, $criteria, 'success', $data,20);
    }

    public function actionFailed($school = '',$grade = '',$class = '',$courseid = '',$keywords = '') {
        set_cookie('_currentUrl_', Yii::app()->request->url);
        $model = Userinfo::model();
        $criteria = new CDbCriteria;
        $data = array();
        $data['coursename']=ClubNews::model()->find("id=".$courseid)->name;
        $criteria->condition=get_where($criteria->condition,$school,'schoolname',$school,'"');
        $criteria->condition=get_where($criteria->condition,$grade,'grade',$grade,'"');
        $criteria->condition=get_where($criteria->condition,$class,'class',$class,'"');
        $criteria->condition=get_like($criteria->condition,'username',$keywords,'');
        $criteria->condition.=' AND id not in (select userid from registration where status=3 and courseid='.$courseid.')'; ////查询指定班级不在报名表中的用户
        parent::_list($model, $criteria, 'failed', $data,20);
    }
```

注意上面最后的sql语句



### 需求二十：退款功能

![](https://s1.ax1x.com/2022/03/23/q3NfSJ.jpg)



![](https://s1.ax1x.com/2022/03/26/qdjbCT.jpg)

#### 子功能一：按订单批量退款

需要实现按下后弹出蒙版

如果是部分退款需要在面板选择按金额/按比例退款

按下后用户确认后退款



此为部分退款

![](https://s1.ax1x.com/2022/03/23/q3UcnI.jpg)

![](https://s1.ax1x.com/2022/03/23/q3UWAf.jpg)

此为全部退款

![](https://s1.ax1x.com/2022/03/23/q3UOEV.jpg)



SignListController.php

```php
    public function actionRefundGroup() {
        set_cookie('_currentUrl_', Yii::app()->request->url);
        $model = ClubNews::model();  //属于ClubNews的model
        $criteria = new CDbCriteria;
        $data = array();
        $criteria->condition=get_where('1=1',"通过",'approvalStatus',"通过",'"');
        $criteria->order = 'id DESC';
        if($_SESSION['F_ROLENAME']==='服务机构') $criteria->condition=get_where($criteria->condition,$_SESSION['TUNIT_id'],'serviceClub',$_SESSION['TUNIT_id'],'"');
        parent::_list($model, $criteria, 'RefundGroup', $data,20);
    }

    public function actionRefundList($courseid) {
        $model = SignList::model();  //属于ClubNews的model
        $criteria = new CDbCriteria;
        $criteria->condition=get_where('1=1',3,'status',3,'"');
        $criteria->condition=get_where($criteria->condition,$courseid,'courseid',$courseid,'"');
        $data=$model->findAll($criteria);
        echo CJSON::encode($data);
    }

    public function actionRefundList2($str) {
        $model = SignList::model();  //属于ClubNews的model
        $criteria = new CDbCriteria;
        $criteria->condition='1=1 AND FIND_IN_SET(id,"'.$str.'")>0';
        $data=$model->findAll($criteria);
        echo CJSON::encode($data);
    }

   public  function actiontgReverse($upOrderId,$lowOrderId,$refundMoney)
    {
        $p = new tgReverse();
        $res = $p->reverse($upOrderId, $lowOrderId);
        $res = json_decode($res);
        if(!empty($res->state))
        {
            if($res->state==5)
            {
                $order = SignList::model()->find("id=".$lowOrderId);
                $order->status=5;
                $order->refundMoney=$refundMoney;
                $order->refundStatus=$res->message;
                if($order->isCoupon==1)
                {
                    $coupon = Coupon::model()->find("f_key='".$order->CouponKey."'");
                    $coupon->isUsed=0;
                    $coupon->orderid=null;
                    $coupon->save();
                }
                $order->save();
                $order->NumberDelete();
                echo CJSON::encode($res);
            }
            else
            {
                echo CJSON::encode($res);
            }
        }
        else
        {
            echo CJSON::encode($res);
        }
    }

    public  function actiontgReverse1($upOrderId,$lowOrderId,$refundMoney) //部分退款
    {
        $p = new tgReverse1();
        $res = $p->reverse($upOrderId,$refundMoney);
        $res = json_decode($res);
        if(!empty($res->state))
        {
            if($res->state==6)
            {
                $order = SignList::model()->find("id=".$lowOrderId);
                $order->status=6;
                $order->refundMoney=$refundMoney;
                $order->refundStatus=$res->message;
                if($order->isCoupon==1)
                {
                    $coupon = Coupon::model()->find("f_key='".$order->CouponKey."'");
                    $coupon->isUsed=0;
                    $coupon->orderid=null;
                    $coupon->save();
                }
                $order->save();
                $order->NumberDelete();
                echo CJSON::encode($res);
            }
            else
            {
                echo CJSON::encode($res);
            }
        }
        else
        {
            echo CJSON::encode($res);
        }
    }


//以下为测试用的退款接口

    public  function actiontgReversetest($lowOrderId='123',$upOrderId='123',$refundMoney='0')
    {
        put_msg($refundMoney);
        $res['upOrderId']=$upOrderId;
        $res['lowOrderId']=$lowOrderId;
        $res['refundMoney'] = $refundMoney;
        $res['message']="成功";
        echo CJSON::encode($res);
    }
}
```



SignList/index (按订单批量退款 + 单个订单退款)

```php+HTML
<head>
    <?php $cs = Yii::app()->clientScript;
    $js_path=Yii::app()->request->baseUrl.'/static/admin';
    $cs->registerCssFile($js_path.'/js/layui/css/layui.css');?>
    <style>
        #masking-bg{background-color: #fffdfd; width: 100%; height: 100%; left: 0; top: 0; z-index: 1; position: fixed; filter: alpha(opacity=80); opacity: 0.8;display: none;}
        #masking-layer{display: none;height:500px;width:100%;position:absolute;top:0;}
        #masking-layer div{position: relative;z-index: 1000;}
        .close{left:80%;top:15%;z-index:1001 !important;}
        .close img{width:30px;}
        .masking-banner{text-align:center;width:100%;}
        .masking-banner .img-bg{width:100%;}
        .masking-banner .img {position:absolute;z-index:1001;left:40%;top:35%;width:25%;}
        .masking-banner div{position:absolute;z-index:1001;left:25%;top:70%;width:50%;}
        .skip{left:38%; top:50%; color:#fff; text-align:center;width:100%;}
        .index-sure{background-color:#2078c7;color:#FFF;padding:10px 20px;text-align: center;border-radius:5px;display:block;width:25%;cursor: pointer;}
    </style>
    <style>
        #masking-bg-Part{background-color: #fffdfd; width: 100%; height: 100%; left: 0; top: 0; z-index: 1; position: fixed; filter: alpha(opacity=80); opacity: 0.8;display: none;}
        #masking-layer-Part{display: none;height:500px;width:100%;position:absolute;top:0;}
        #masking-layer-Part div{position: relative;z-index: 1000;}
        .close{left:80%;top:15%;z-index:1001 !important;}
        .close img{width:30px;}
        .masking-banner{text-align:center;width:100%;}
        .masking-banner .img-bg{width:100%;}
        .masking-banner .img {position:absolute;z-index:1001;left:40%;top:35%;width:25%;}
        .masking-banner div{position:absolute;z-index:1001;left:25%;top:70%;width:50%;}
        .skip{left:38%; top:50%; color:#fff; text-align:center;width:100%;}
        .index-sure{background-color:#2078c7;color:#FFF;padding:10px 20px;text-align: center;border-radius:5px;display:block;width:25%;cursor: pointer;}
    </style>
    <style>
        #masking-bg-Part-single{background-color: #fffdfd; width: 100%; height: 100%; left: 0; top: 0; z-index: 1; position: fixed; filter: alpha(opacity=80); opacity: 0.8;display: none;}
        #masking-layer-Part-single{display: none;height:500px;width:100%;position:absolute;top:0;}
        #masking-layer-Part-single div{position: relative;z-index: 1000;}
        .close{left:80%;top:15%;z-index:1001 !important;}
        .close img{width:30px;}
        .masking-banner{text-align:center;width:100%;}
        .masking-banner .img-bg{width:100%;}
        .masking-banner .img {position:absolute;z-index:1001;left:40%;top:35%;width:25%;}
        .masking-banner div{position:absolute;z-index:1001;left:25%;top:70%;width:50%;}
        .skip{left:38%; top:50%; color:#fff; text-align:center;width:100%;}
        .index-sure{background-color:#2078c7;color:#FFF;padding:10px 20px;text-align: center;border-radius:5px;display:block;width:25%;cursor: pointer;}
    </style>
</head>
<div class="box">
    <?php echo show_title($this)?>
    <div class="box-content">
        <div class="box-header">
            <a class="btn" href="javascript:;" onclick="we.reload();"><i class="fa fa-refresh"></i>刷新</a>
            <a style="display:none;" id="j-delete" class="btn" href="javascript:;" onclick="we.dele(we.checkval('.check-item input:checked'), deleteUrl);"><i class="fa fa-trash-o"></i>刪除</a>
            <a style="display:none;" id="jR" class="btn" href="javascript:;" onclick="showMasking(we.checkval('.check-item input:checked'));"><i class="fa fa-reply-all"></i>批量退款</a>
            <a style="display:none;" id="jRS" class="btn" href="javascript:;" onclick="showPartMasking(we.checkval('.check-item input:checked'));"><i class="fa fa-reply"></i>批量部分退款</a>
        </div><!--box-header end-->


        <div class="box-search">
            <form action="<?php echo Yii::app()->request->url;?>" method="get">
                <input type="hidden" name="r" value="<?php echo Yii::app()->request->getParam('r');?>">
                <input type="hidden" name="id" id="id" value="<?php echo $_REQUEST['id'];?>">
                <span>活动名称</span>
                <input style="width:200px;" class="input-text" id="course_name" type="text" readonly='ture'>
                <input type="hidden" style="width:200px;" id="course_id" class="input-text" type="text" name="courseid">

                <input id="course_select_btn" class="btn" type="button" value="选择">
                <label style="margin-right:10px;">
                    <span>任意订单号查询：</span>
                    <input style="width:200px;" class="input-text" type="text" name="keywords" value="<?php echo Yii::app()->request->getParam('keywords');?>">
                </label>
                <button class="btn btn-blue" type="submit">查询</button>
            </form>

            <!--  <td><//?php echo $form->labelEx($model, 'courseid'); ?></td>
                 <td>
                  <//?php echo $form->textField($model, 'courseid', array('class' => 'input-text-add','readonly'=>'ture')); ?>
                     <input id="club_select_btn" class="btn" type="button" value="选择">
                     <//?php echo $form->error($model, 'courseid', $htmlOptions = array()); ?>
                 </td> -->
        </div><!--box-search end-->

        <?php if($_REQUEST['id']==1){?>
        <div class="box-table">
            <table class="list">
                <thead>

                <tr>
                    <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
                    <th style='text-align: center;'>编号</th>
                    <th style='text-align: center;'>用户姓名</th>
                    <th style='text-align: center;'>报名活动</th>
                    <th style='text-align: center;'>报名时间</th>
                    <th style='text-align: center;'>支付金额</th>
                    <th style='text-align: center;'>支付截止时间</th>
                    <th style='text-align: center;'>状态</th>
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
                        <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
                        <td style='text-align: center;'><?php echo $v->username; ?></td>
                        <td style='text-align: center;'><?php echo $v->coursename; ?></td>
                        <td style='text-align: center;'><?php echo $v->registrationtime; ?></td>
                        <td style='text-align: center;'><?php echo $v->payMoney; ?></td>
                        <td style='text-align: center;'><?php echo $v->payendTime; ?></td>
                        <td style='text-align: center;'><?php echo '<span style="color: red">'."报名不成功</span>" ?></td>
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
<?php } else if ($_REQUEST['id']==2){?>
    <div class="box-table">
        <table class="list">
            <thead>

            <tr>
                <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
                <th style='text-align: center;'>编号</th>
                <th style='text-align: center;'>用户姓名</th>
                <th style='text-align: center;'>报名活动</th>
                <th style='text-align: center;'>报名时间</th>
                <th style='text-align: center;'>支付金额</th>
                <th style='text-align: center;'>支付截止时间</th>
                <th style='text-align: center;'>状态</th>
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
                    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
                    <td style='text-align: center;'><?php echo $v->username; ?></td>
                    <td style='text-align: center;'><?php echo $v->coursename; ?></td>
                    <td style='text-align: center;'><?php echo $v->registrationtime; ?></td>
                    <td style='text-align: center;'><?php echo $v->payMoney; ?></td>
                    <td style='text-align: center;'><?php echo $v->payendTime; ?></td>
                    <td style='text-align: center;'><?php echo '报名未付款' ?></td>
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
<?php } else if ($_REQUEST['id']==3){?>
    <div class="box-table">
        <table class="list">
            <thead>

            <tr>
                <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
                <th style='text-align: center;'>编号</th>
                <th style='text-align: center;'>用户姓名</th>
                <th style='text-align: center;'>报名活动</th>
                <th style='text-align: center;'>报名时间</th>
                <th style='text-align: center;'>支付渠道</th>
                <th style='text-align: center;'>平台订单号</th>
                <th style='text-align: center;'>第三方订单号</th>
                <th style='text-align: center;'>微信订单号</th>
                <th style='text-align: center;'>支付金额</th>
                <th style='text-align: center;'>支付截止时间</th>
                <th style='text-align: center;'>状态</th>
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
                    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
                    <td style='text-align: center;'><?php echo $v->username; ?></td>
                    <td style='text-align: center;'><?php echo $v->coursename; ?></td>
                    <td style='text-align: center;'><?php echo $v->registrationtime; ?></td>
                    <td style='text-align: center;'><?php echo $v->paychannel; ?></td>
                    <td style='text-align: center;'><?php echo $v->id; ?></td>
                    <td style='text-align: center;'><?php echo $v->payid; ?></td>
                    <td style='text-align: center;'><?php echo $v->wxpayid; ?></td>
                    <td style='text-align: center;'><?php echo $v->payMoney; ?></td>
                    <td style='text-align: center;'><?php echo $v->payendTime; ?></td>
                    <td style='text-align: center;'><?php echo '<span style="color: green">'."报名成功</span>" ?></td>
                    <td style='text-align: center;'>
                        <a class="btn" href="javascript:;" onclick="we.dele('<?php echo $v->id;?>', deleteUrl);" title="删除"><i class="fa fa-trash-o"></i></a>
                        <button class="btn" onclick='reverse(this)' type="button" down="<?php echo $v->id; ?>" up="<?php echo $v->payid; ?>" title="退款"><i class="fa fa-undo">退款</i></button>
                        <button class="btn" onclick='reversePart(this)' type="button" down="<?php echo $v->id; ?>" up="<?php echo $v->payid; ?>" title="退款"><i class="fa fa-undo">部分退款</i></button>
                    </td>
                </tr>
                <?php $index++; } ?>
            </tbody>
        </table>
    </div><!--box-table end-->
    <div class="box-page c"><?php $this->page($pages);?></div>

    </div><!--box-content end-->
    </div><!--box end-->
<?php } else if ($_REQUEST['id']==4){?>
    <div class="box-table">
        <table class="list">
            <thead>

            <tr>
                <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
                <th style='text-align: center;'>编号</th>
                <th style='text-align: center;'>用户姓名</th>
                <th style='text-align: center;'>报名活动</th>
                <th style='text-align: center;'>报名时间</th>
                <th style='text-align: center;'>支付渠道</th>
                <th style='text-align: center;'>平台订单号</th>
                <th style='text-align: center;'>第三方订单号</th>
                <th style='text-align: center;'>微信订单号</th>
                <th style='text-align: center;'>支付金额</th>
                <th style='text-align: center;'>支付截止时间</th>
                <th style='text-align: center;'>状态</th>
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
                    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
                    <td style='text-align: center;'><?php echo $v->username; ?></td>
                    <td style='text-align: center;'><?php echo $v->coursename; ?></td>
                    <td style='text-align: center;'><?php echo $v->registrationtime; ?></td>
                    <td style='text-align: center;'><?php echo $v->paychannel; ?></td>
                    <td style='text-align: center;'><?php echo $v->id; ?></td>
                    <td style='text-align: center;'><?php echo $v->payid; ?></td>
                    <td style='text-align: center;'><?php echo $v->wxpayid; ?></td>
                    <td style='text-align: center;'><?php echo $v->payMoney; ?></td>
                    <td style='text-align: center;'><?php echo $v->payendTime; ?></td>
                    <td style='text-align: center;'><?php echo '<span style="color: orange">'."发起退款申请</span>" ?></td>
                    <td style='text-align: center;'>
                        <a class="btn" href="javascript:;" onclick="we.dele('<?php echo $v->id;?>', deleteUrl);" title="删除"><i class="fa fa-trash-o"></i></a>
                        <button class="btn" onclick='reverse(this)' type="button" down="<?php echo $v->id; ?>" up="<?php echo $v->payid; ?>" RMONEY="<?php echo $v->payMoney; ?>" title="同意退款"><i class="fa fa-check" style="color:green">同意</i></button>
                        <button class="btn" onclick='reversePart(this)' type="button" down="<?php echo $v->id; ?>" up="<?php echo $v->payid; ?>" RMONEY="<?php echo $v->payMoney; ?>" title="部分退款"><i>部分退款</i></button>
                        <button class="btn" onclick='reject(this)' type="button" down="<?php echo $v->id; ?>" up="<?php echo $v->payid; ?>" title="拒绝退款"><i class="fa fa-times" style="color:red">拒绝</i></button>
                    </td>
                </tr>
                <?php $index++; } ?>
            </tbody>
        </table>
    </div><!--box-table end-->
    <div class="box-page c"><?php $this->page($pages);?></div>

    </div><!--box-content end-->
    </div><!--box end-->
<?php } else if ($_REQUEST['id']==5){?>
    <div class="box-table">
        <table class="list">
            <thead>

            <tr>
                <th class="check"><input id="j-checkall" class="input-check" type="checkbox"></th>
                <th style='text-align: center;'>编号</th>
                <th style='text-align: center;'>用户姓名</th>
                <th style='text-align: center;'>报名活动</th>
                <th style='text-align: center;'>报名时间</th>
                <th style='text-align: center;'>支付渠道</th>
                <th style='text-align: center;'>平台订单号</th>
                <th style='text-align: center;'>第三方订单号</th>
                <th style='text-align: center;'>微信订单号</th>
                <th style='text-align: center;'>支付金额</th>
                <th style='text-align: center;'>支付截止时间</th>
                <th style='text-align: center;'>状态</th>
                <th style='text-align: center;'>退款金额</th>
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
                    <td style='text-align: center;'><span class="num num-1"><?php echo $index ?></span></td>
                    <td style='text-align: center;'><?php echo $v->username; ?></td>
                    <td style='text-align: center;'><?php echo $v->coursename; ?></td>
                    <td style='text-align: center;'><?php echo $v->registrationtime; ?></td>
                    <td style='text-align: center;'><?php echo $v->paychannel; ?></td>
                    <td style='text-align: center;'><?php echo $v->id; ?></td>
                    <td style='text-align: center;'><?php echo $v->payid; ?></td>
                    <td style='text-align: center;'><?php echo $v->wxpayid; ?></td>
                    <td style='text-align: center;'><?php echo $v->payMoney; ?></td>
                    <td style='text-align: center;'><?php echo $v->payendTime; ?></td>
                    <td style='text-align: center;'><?php echo '<span style="color: green">'.$v->refundStatus ?></td>
                    <td style='text-align: center;'><?php echo $v->refundMoney; ?></td>
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
<?php } ?>

<div id="masking-bg">
</div>
<div id="masking-layer">
    <div class="layui-progress layui-progress-big" lay-showPercent="yes" style="left:15%;top:30%;width:70%;" lay-filter="demo">
        <div class="layui-progress-bar layui-bg-blue" lay-percent="0%"></div>
        <div class="layui-card" style="margin-top:5%;width:100%;">
            <div class="layui-card-header" style="text-align: center">批量退款申请状态明细</div>
            <div class="layui-card-body" style="overflow-y:scroll; height:100px" id="msgbox">
                --------开始批量申请---------<br>
            </div>
        </div>
        <div class="skip" style="margin-top:5%">
            <div class="index-sure font-13" onclick="closeMasking()">
                完成退款
            </div>
        </div>
    </div>
</div>

<!--下面是部分退款的-->
<div id="masking-bg-Part">
</div>
<div id="masking-layer-Part">

    <div class="layui-card" style="left:15%;top:30%;width:70%">
        <div class="layui-card-header" style="text-align: center">批量部分退款设置(操作不可逆)</div>
        <div class="layui-card-body">
                <form class="layui-form"  action="">
                    <div style="width: 300px; display: inline-block; margin-right: 10px;">
                        <select  xm-select="method-value-example1" id="Method">
                            <option value="1">输入退款比例</option>
                            <option value="2">输入退款金额</option>
                        </select>
                    </div>
                </form>
                <button class="layui-btn" style="margin-top: 5px" onclick="onMethod()" id="onMethod">确认部分退款输入方式</button>
            <input type="number" id="inputMoney" style="margin-top: 5px" required lay-verify="required" placeholder="请输入需要部分退款的比例(0-100)" autocomplete="off" class="layui-input" value=""  >
            <input type="number" id="inputMoney2" style="margin-top: 5px" required lay-verify="required" placeholder="请输入需要部分退款的金额/百分比" autocomplete="off" class="layui-input" value=""  >
            <div  style="margin-top:5px;left: 40%;width: 20%"><button class="layui-btn layui-btn-fluid" id="confirm_money" onclick="submitMoney()" style="Display:none">确认退款金额/百分比</button></div>
            <div id="input_money" class="layui-btn layui-btn-radius layui-btn-normal"  style="margin-top:5px;text-align:center;left: 20%;width: 60%;display: none"></div>
        </div>

    </div>
    <div class="skip" style="margin-top:5%">
        <div class="index-sure font-13" id="confirm_refund" onclick="showMasking(we.checkval('.check-item input:checked'))">
            开始退款
        </div>
    </div>
    <div class="skip" style="margin-top:5%">
        <div class="index-sure font-13" id="confirm_refund" onclick="closeMasking()">
            取消
        </div>
    </div>
</div>


<!--下面是单人部分退款的-->
<div id="masking-bg-Part-single">
</div>
<div id="masking-layer-Part-single">

    <div class="layui-card" style="left:15%;top:30%;width:70%">
        <div class="layui-card-header" style="text-align: center">部分退款设置(操作不可逆) 请从退款金额和百分比中择一填写</div>
        <div class="layui-card-body">
            <text><strong>请输入部分退款百分比</strong></text>
            <input type="number" id="inputMoneySingle" required lay-verify="required" placeholder="请输入需要部分退款的比例(0-100)" autocomplete="off" class="layui-input" onchange="input()" value=""  >
            <text><strong>请输入部分退款金额</strong></text>
            <input type="number" id="inputMoneySingle2" required lay-verify="required" placeholder="请输入退款的金额" autocomplete="off" class="layui-input" onchange="input2()" value=""  >
            <div  style="margin-top:5px;left: 40%;width: 20%"><button class="layui-btn layui-btn-fluid" id="confirm_money_single" onclick="submitMoneySingle()" >确认退款</button></div>
            <div id="input_money_single" class="layui-btn layui-btn-radius layui-btn-normal"  style="margin-top:5px;text-align:center;left: 20%;width: 60%;display: none"></div>
        </div>

    </div>
<!--    <div class="skip" style="margin-top:5%">-->
<!--        <div class="index-sure font-13" id="confirm_refund_single" onclick="closeMasking()">-->
<!--            开始退款-->
<!--        </div>-->
</div>


<script>
    var reverse_money = -1; //此为部分退款金额百分比的全局变量 (0-100)
    var Ratio = 0;
    var Money = 0;

    var MoneyInputMethod = 0; // 1 为输入退款百分比 2为输入退款金额

    function onMethod() //确定退款输入方式
    {
        var x = document.getElementById("Method");
        //console.log(x.value);
        if(x.value == 1) {
            MoneyInputMethod = 1;
            $("#inputMoney").show();
            $("#inputMoney2").hide(); //隐藏输入退款金额
            $("#confirm_money").show();
        }
        else if (x.value == 2){
            MoneyInputMethod = 2;
            $("#inputMoney2").hide();
            $("#confirm_money").hide();
        }
    }

    function input() //监听input事件
    {
        var x =document.getElementById("inputMoneySingle");
        Ratio = x.value;
        //console.log(Ratio);
        var money = RMONEY*Ratio*0.01;
        money = money.toFixed(2);
        document.getElementById('inputMoneySingle2').value=money;  //计算钱数回填到金额框中
    }

    function input2() //监听input2事件
    {
        var x =document.getElementById("inputMoneySingle2");
        Money = x.value;
        //console.log(Money);
        var ratio = Money/RMONEY*100;
        ratio = ratio.toFixed(2);
        Ratio =ratio;
        document.getElementById('inputMoneySingle').value=ratio;  //计算钱数回填到金额框中
    }


    $(function() {
        //$("html").niceScroll({zindex: '1000'});
//console.log('public row =5');
        var $jDelete = $('#jR');
        // console.log('public length ='+$jDelete.length);
        if ($jDelete.length > 0) {
            var $this, $temp1 = $('.check-item .input-check'), $temp2 = $('.box-table .list tbody tr');

            $('#j-checkall').on('click', function() {
                $this = $(this);
                if ($this.is(':checked')) {
                    $temp1.each(function() {
                        this.checked = true;
                    });
                    $temp2.addClass('selected');
                } else {
                    $temp1.each(function() {
                        this.checked = false;
                    });
                    $temp2.removeClass('selected');
                }
                we.hasDelete('.check-item .input-check:checked', '#jR');
            });

            $temp1.each(function() {
                $this = $(this);
                if ($this.is(':checked')) {
                    $this.parent().parent().addClass('selected');
                } else {
                    $this.parent().parent().removeClass('selected');
                }
            });

            $temp1.on('click', function() {
                $this = $(this);
                if ($this.is(':checked')) {
                    $this.parent().parent().addClass('selected');
                } else {
                    $this.parent().parent().removeClass('selected');
                }
                we.hasDelete('.check-item .input-check:checked', '#jR');
            });
        }
    });

    $(function() {
        //$("html").niceScroll({zindex: '1000'});
//console.log('public row =5');
        var $jDelete = $('#jRS');
        // console.log('public length ='+$jDelete.length);
        if ($jDelete.length > 0) {
            var $this, $temp1 = $('.check-item .input-check'), $temp2 = $('.box-table .list tbody tr');

            $('#j-checkall').on('click', function() {
                $this = $(this);
                if ($this.is(':checked')) {
                    $temp1.each(function() {
                        this.checked = true;
                    });
                    $temp2.addClass('selected');
                } else {
                    $temp1.each(function() {
                        this.checked = false;
                    });
                    $temp2.removeClass('selected');
                }
                we.hasDelete('.check-item .input-check:checked', '#jRS');
            });

            $temp1.each(function() {
                $this = $(this);
                if ($this.is(':checked')) {
                    $this.parent().parent().addClass('selected');
                } else {
                    $this.parent().parent().removeClass('selected');
                }
            });

            $temp1.on('click', function() {
                $this = $(this);
                if ($this.is(':checked')) {
                    $this.parent().parent().addClass('selected');
                } else {
                    $this.parent().parent().removeClass('selected');
                }
                we.hasDelete('.check-item .input-check:checked', '#jRS');
            });
        }
    });



    $('#masking-layer').height($(window).height());
    var deleteUrl = '<?php echo $this->createUrl('delete', array('id'=>'ID'));?>';
    $('#course_select_btn').on('click', function(){ read_course(); });

    function read_course(){
        $.dialog.data('id', 0);
        //console.log($.dialog.data('id'));
        $.dialog.open('<?php echo $this->createUrl("select/course");?>',{
            id:'course',lock:true,opacity:0.3,width:'500px',height:'60%',
            title:'选择活动',
            close: function () {
                //console.log($.dialog.data('id'));
                if($.dialog.data('id')>0){
                    $('#course_name').val($.dialog.data('name'));
                    $('#course_id').val($.dialog.data('id'));
                }
            }
        });
    }

    function read_score(sid){
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

    function reverse(element){   //此为单条订单全部退款 js
        var url = "<?php echo $this->createUrl('tgReversetest', array('upOrderId'=>'UPSID','lowOrderId'=>'LOWSID','refundMoney'=>'RMONEY'));?>";
        var upsid= element.getAttribute("up");
        var lowsid= element.getAttribute("down");
        var RMONEY= element.getAttribute("rmoney");
        //console.log(element);
        url = url.replace(/UPSID/, upsid);
        url = url.replace(/LOWSID/, lowsid);
        url = url.replace(/RMONEY/, RMONEY);
        if(confirm("确定要为此订单退款吗？")){
            $.ajax({
                url: url,
                type: "get",
                dataType : 'json',
                success: function(res) {
                    alert(res.message);
                    location.reload();
                }
            });
        }
        else{
            //alert("取消了退款");
        }
    }
    var url;
    var upsid;
    var lowsid;
    var RMONEY;
    function reversePart(element)  //此为单条订单部分退款 js
    {
        url = "<?php echo $this->createUrl('tgReversetest', array('upOrderId'=>'UPSID','lowOrderId'=>'LOWSID','refundMoney'=>'RMONEY'));?>";
        upsid= element.getAttribute("up");
        lowsid= element.getAttribute("down");
        RMONEY= element.getAttribute("rmoney");
        //以下为填写退款比例
        if(confirm("确定要为此订单退款吗？")){
            $('#masking-bg-Part-single').css('display', 'block');
            $('#masking-layer-Part-single').css('display', 'block');
            console.log(reverse_money);
        }
        else{
            //alert("取消了退款");
        }
    }

    function reject(element){
        var url = "<?php echo $this->createUrl('reject', array('id'=>'LOWSID'));?>";
        var lowsid= element.getAttribute("down");
        url = url.replace(/LOWSID/, lowsid);
        $.ajax({
            url: url,
            type: "get",
            success: function(res) {
                location.reload();
            }
        });
    }



    function showMasking(str) {
        if(confirm("确定为选择的订单退款吗？")){
            $("#confirm_money").attr("style","display:none;");
            // $("#Method").attr("style","display:none;");
            // $("#onMethod").attr("style","display:none;");
            $("#inputMoney").attr("style","display:none;");
            $("#inputMoney2").attr("style","display:none;");
            $("#confirm_refund").attr("style","display:none;");
            //以上代码为确认退款后禁用修改或再次退款
            $('#masking-bg').css('display', 'block');
            $('#masking-layer').css('display', 'block');
            //console.log(1000);
            $.ajax({    //获取勾选的订单详细数据
                url: "<?php echo $this->createUrl("SignList/RefundList2")?>",
                data: {
                    "str": str
                },
                type: "GET",
                dataType: "json",
                success: function (data) {  //如果成功
                    console.log(data);
                    var cnt=data.length;    //计算该课程人数
                    var n=0;    //退款任务已执行人数初始化0
                    var p;  //比例
                    if(reverse_money === -1) {
                        for (let i = 0; i < cnt; i++) { //对每个订单进行退款
                            setTimeout(function () {
                                $.ajax({

                                    url: "<?php echo $this->createUrl("SignList/tgReversetest")?>", //测试端口：tgReversetest
                                    data: {
                                        "upOrderId": data[i].payid,
                                        "lowOrderId": data[i].id
                                    },
                                    type: "GET",
                                    dataType: "json",
                                    success: function (res) {
                                        n = n + 1;  //退款任务已执行人数+1
                                        p = n / cnt * 100;    //计算比例
                                        p = p.toFixed(2); //保留两位小数
                                        document.getElementById("msgbox").innerHTML += res.lowOrderId + ":" + res.message + "<br>"; //写入消息框
                                        layui.use('element', function () {
                                            var $ = layui.jquery
                                                , element = layui.element; //Tab的切换功能，切换事件监听等，需要依赖element模块
                                            element.progress('demo', p + '%');  //进度条变化
                                        });
                                    },
                                    error: function () {
                                        document.getElementById("msgbox").innerHTML += "出现错误" + "<br>"; //写入消息框
                                    }

                                });
                            }, 10 * i);   //0.01秒执行一条
                        }
                    }else {
                        for (let i = 0; i < cnt; i++) { //对每个订单进行退款
                            let tmpMoney = 0;
                            if(MoneyInputMethod == 1) {  //输入方式为百分比
                                tmpMoney = reverse_money; //此处必须用let 作用域为for循环内，而var作用域是整个函数快，导致无法更新
                                tmpMoney = tmpMoney * 0.01 * data[i].payMoney; //乘以退款比例
                            }
                            else if (MoneyInputMethod == 2){
                                tmpMoney = reverse_money;
                            }
                            console.log(tmpMoney);
                            setTimeout(function () {
                                $.ajax({

                                    url: "<?php echo $this->createUrl("SignList/tgReversetest")?>", //测试端口：tgReversetest
                                    data: {
                                        "upOrderId": data[i].payid,
                                        "lowOrderId": data[i].id,
                                        "refundMoney":tmpMoney
                                    },
                                    type: "GET",
                                    dataType: "json",
                                    success: function (res) {
                                        n = n + 1;  //退款任务已执行人数+1
                                        p = n / cnt * 100;    //计算比例
                                        p = p.toFixed(2); //保留两位小数
                                        document.getElementById("msgbox").innerHTML += res.lowOrderId + ":" + res.message + "<br>"; //写入消息框
                                        layui.use('element', function () {
                                            var $ = layui.jquery
                                                , element = layui.element; //Tab的切换功能，切换事件监听等，需要依赖element模块
                                            element.progress('demo', p + '%');  //进度条变化
                                        });
                                    },
                                    error: function () {
                                        document.getElementById("msgbox").innerHTML += "出现错误" + "<br>"; //写入消息框
                                    }

                                });
                            }, 10 * i);   //0.01秒执行一条
                        }
                    }
                }
            });

        }
        else{
            //alert("取消了退款");
        }
    }

    function submitMoney(){
        //console.log(111);
        if(MoneyInputMethod === 1)
            reverse_money = $('#inputMoney').val();  //这里传入部分退款的百分比
        else if(MoneyInputMethod === 2)
            reverse_money = $('#inputMoney2').val();  //这里传入部分退款的金额
        //console.log(reverse_money);
            if ((reverse_money <= 0 || reverse_money > 100) && MoneyInputMethod === 1) //防止出现负数金额
            {
                alert("您输入的百分比无效");
            }
        else {
            $("#confirm_refund").attr("style","display:true;");
            if(MoneyInputMethod === 1) {
                $("#input_money").show();
                document.getElementById('input_money').innerHTML = "部分退款的百分比是：" + reverse_money + "%";
            }
            else{
                $("#input_money").show();
                document.getElementById('input_money').innerHTML="部分退款的金额是："+reverse_money+"元";
            }
            //console.log(100000);
            //console.log(reverse_money);
        }
    }

    function submitMoneySingle(){
        reverse_money = $('#inputMoneySingle2').val();  //这里传入部分退款的金额

        //console.log(reverse_money);
        if(Ratio <= 0 || Ratio > 100) //防止出现负数金额
        {
            alert("您输入的百分比无效");
        }
        else {
            $("#confirm_refund_single").attr("style","display:true;");
            $("#input_money_single").show();
            document.getElementById('input_money_single').innerHTML="部分退款的金额是："+reverse_money;
            if(reverse_money != -1) {
                //console.log(reverse_money);
                RMONEY = reverse_money;
               // console.log("reverse_money");
               // console.log(reverse_money);
                url = url.replace(/UPSID/, upsid);
                url = url.replace(/LOWSID/, lowsid);
                url = url.replace(/RMONEY/, RMONEY);
                //console.log(RMONEY);
                $.ajax({
                    url: url,
                    type: "get",
                    dataType : 'json',
                    success: function(res) {
                        alert(res.message);
                        location.reload();
                    }
                });
            }
            //console.log(100000);
            //console.log(reverse_money);
        }
    }

    function showPartMasking(str) { //部分退款的页面填写 js
        if(confirm("请填写部分退款的百分比")){
            $("#confirm_refund").attr("style","display:none;");
            $("#inputMoney2").attr("style","display:none;");
            $("#inputMoney").attr("style","display:none;");

            $('#masking-bg-Part').css('display', 'block');
            $('#masking-layer-Part').css('display', 'block');
           // showPartMasking2(str);
        }
        else{
            //alert("取消了退款");
        }

    }



    function closeMasking() {
        //console.log(reverse_money);
        $('#masking-bg').css('display', 'none');
        $('#masking-layer').css('display', 'none');
        location.reload();
    }


</script>


```





**技术实现细节**

1.隐藏或展示一个组件

```html
<input type="number" id="inputMoney" style="margin-top: 5px" required lay-verify="required" placeholder="请输入需要部分退款的比例(0-100)" autocomplete="off" class="layui-input" value=""  >
```

 如需要隐藏或展示 inputMoney这个按钮

```js
$("#inputMoney").hide();
$("#inputMoney").show();
```



2.js获取按钮提交的值

```html
<div  style="margin-top:5px;left: 40%;width: 20%"><button class="layui-btn layui-btn-fluid" id="confirm_money" onclick="submitMoney()" style="Display:none">确认退款金额/百分比</button></div>
```

```js
reverse_money = $('#inputMoney').val();
```

3.在HTML的某个标签处插入显示ｊｓ的变量值

```html
<div id="input_money" class="layui-btn layui-btn-radius layui-btn-normal"  style="margin-top:5px;text-align:center;left: 20%;width: 60%;display: none"></div>
```

```js
document.getElementById('input_money').innerHTML = "部分退款的百分比是：" + reverse_money + "%";
```

4.单个订单退款时提供两个输入框，输入百分比时自动同步计算金额，输入金额也一样

```html
            <input type="number" id="inputMoneySingle" required lay-verify="required" placeholder="请输入需要部分退款的比例(0-100)" autocomplete="off" class="layui-input" onchange="input()" value=""  >
            <text><strong>请输入部分退款金额</strong></text>
            <input type="number" id="inputMoneySingle2" required lay-verify="required" placeholder="请输入退款的金额" autocomplete="off" class="layui-input" onchange="input2()" value=""  >
```

```js
    var reverse_money = -1; //此为部分退款金额百分比的全局变量 (0-100)
    var Ratio = 0;
    var Money = 0;

function input() //监听input事件
    {
        var x =document.getElementById("inputMoneySingle");
        Ratio = x.value;
        //console.log(Ratio);
        var money = RMONEY*Ratio*0.01;
        money = money.toFixed(2); //保留2位小数
        document.getElementById('inputMoneySingle2').value=money;  //计算钱数回填到金额框中
    }

    function input2() //监听input2事件
    {
        var x =document.getElementById("inputMoneySingle2");
        Money = x.value;
        //console.log(Money);
        var ratio = Money/RMONEY*100;
        ratio = ratio.toFixed(2);
        Ratio =ratio;
        document.getElementById('inputMoneySingle').value=ratio;  //计算钱数回填到金额框中
    }
```

5.注意：js语言代码  如果只需要变量作用域在for循环里面 则 要使用  let 定义  因为var在块 {} 外也能访问

```js
                        for (let i = 0; i < cnt; i++) { //对每个订单进行退款
                            let tmpMoney = 0;
                            if(MoneyInputMethod == 1) {  //输入方式为百分比
                                tmpMoney = reverse_money; //此处必须用let 作用域为for循环内，而var作用域是整个函数快，导致无法更新
                                tmpMoney = tmpMoney * 0.01 * data[i].payMoney; //乘以退款比例
                            }
                            else if (MoneyInputMethod == 2){
                                tmpMoney = reverse_money;
                            }
```

6.xm-select 配合 layUi 的下拉选框

```html
                <form class="layui-form"  action="">
                    <div style="width: 300px; display: inline-block; margin-right: 10px;">
                        <select  xm-select="method-value-example1" id="Method">
                            <option value="1">输入退款比例</option>
                            <option value="2">输入退款金额</option>
                        </select>
                    </div>
                </form>
```



#### 子功能二：单个订单退款

代码在上面

实现提供两个输入框，输入百分比时自动同步计算金额，输入金额也一样

![](https://s1.ax1x.com/2022/03/24/qGoHqs.jpg)















### 需求二十一：navicat MySQL 定时任务

首先检查是否开启定时事件功能  若为ON则启动   电脑关机会关闭

```sql
show variables like '%sche%';
```

若未启动则运行以下 sql 语句

```sql
set global event_scheduler =1
```

![](https://s1.ax1x.com/2022/03/25/qtSBkT.jpg)

![](https://s1.ax1x.com/2022/03/25/qtSLnI.png)

![](https://s1.ax1x.com/2022/03/25/qtpOxJ.png)

以上sql语句功能是从从registration表里面找到和course表里面每个 id  想同的courseid的数量 并赋值给course表对应的sign_num



### 需求二十二  给每个研学班级分配学校教师

需求：导出分组名单时一起导出，查询每个班级已报名该课程的学校教师

![](https://s1.ax1x.com/2022/03/27/qw8ANn.png)

为了把时间复杂度从 n^2降低  考虑从教师报名表里面遍历把对应的教师存到班级表里面（作为哈希表）然后遍历学生报名表查找对应班级把教师字符串填入

```php
   public function actionExcel($id,$name){
        //以下是给每个班填入学校老师
        $class = BaseClass::model()->findAll();
        foreach ($class as $v)
        {
            $schoolname = $v->school;
            $grade = $v->grade;
            $class = $v->class;
            $criteriaTeacher = new CDbCriteria();
            $criteriaTeacher->condition=get_where('1=1',$schoolname,'schoolname',$schoolname,'"');
            $criteriaTeacher -> condition = get_where($criteriaTeacher -> condition,$grade,'grade',$grade);
            $criteriaTeacher -> condition = get_where($criteriaTeacher -> condition,$class,'class',$class);
            $criteriaTeacher -> condition = get_where($criteriaTeacher -> condition,$id,'courseid',$id);
            $criteriaTeacher -> condition = get_where($criteriaTeacher -> condition,2,'type',2);
            $teachers = teaSignList::model()->findAll($criteriaTeacher);
            $teacherString = "";
            foreach ($teachers as $j)
            {
                $teacherString.=$j->username.=",";
            }
            $v->teacher = $teacherString;
            $v->save();
        }

        //给每个该课程的学生填入学校老师
        $students = SignList::model()->findAll('courseid='.$id);
        foreach ($students as $v)
        {
            $school = $v->school;
            $grade = $v->grade;
            $class = $v->class;
            $criteriaClass = new CDbCriteria();
            $criteriaClass->condition=get_where('1=1',$school,'school',$school,'"');
            $criteriaClass -> condition = get_where($criteriaClass -> condition,$grade,'grade',$grade);
            $criteriaClass -> condition = get_where($criteriaClass -> condition,$class,'class',$class);
            $Class = BaseClass::model()->find($criteriaClass);
            $v->school_teacher = $Class->teacher;
            //put_msg($Class->teacher);
            $v->save();
        }

        //先定义一个excel文件
        $filename   =   date('【分组结果总表】('.date('Y-m-d H:i:s').'导出)').".xls";
        header("Content-Type: application/vnd.ms-execl");
        header("Content-Type: application/vnd.ms-excel; charset=utf-8");
        header("Content-Disposition: attachment; filename=$filename");
        header("Pragma: no-cache");
        header("Expires: 0");
        //条件
        $criteria = new CDbCriteria() ;  //新建筛选
        $criteria -> select = array('id','userid','username','sex','grade','class','status','teacher','groupnum','car_id','car','room_id','room','school_teacher');   //筛选字段
        $criteria -> condition = ('1=1');    //筛选还没被分组的名单，即字段group=0
        $criteria -> condition = get_where($criteria -> condition,$id,'courseid',$id);  //筛选课程（可传递参数）
        $criteria -> condition = get_where($criteria -> condition,3,'status',3,'"');    //筛选成功报名的名单
        $criteria -> order = 'groupnum asc,grade asc,class asc';   //指定排序

        $criteria_tea = new CDbCriteria();




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
                                        <th>学校导师</th>
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
        $criteria1 -> condition = get_where("1=1",$id,'courseid',$id);
        $g_num = Group::model()->findALL($criteria1);
        //html语句填充数据
        if(empty($search)){}
        else{
            $i=0;
            $j=0;
            foreach ($search as $k) {
                $excel_content  .= '<td>'.$k->username.'</td>';
                $excel_content  .= '<td>'.($k->sex).'</td>';
                $excel_content  .= '<td>'.$k->grade.'</td>';
                $excel_content  .= '<td>'.$k->class.'</td>';
                if($j==0)
                {
                    $temp=count(SignList::model()->findAll("groupnum=".$g_num[$i]->g_id." AND courseid=".$id));
                    $excel_content  .= '<td style="text-align: center;" rowspan='.$temp.'>'.$k->groupnum.'</td>';
                    $j++;$i++;
                }
                else if($j<$temp-1) $j++;
                else $j=0;
                $excel_content  .= '<td>'.$k->teacher.'</td>';
                $excel_content  .= '<td>'.$k->school_teacher.'</td>';
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

