---
layout: post
title:  "Node 로그인인증 구현"
date : 2018-03-13 12:46:32 +0900
description: 
categories: Node AWS-S3
tags: Node AWS-S3
---

# Amazon S3란??

## HTML Form 
{% highlight HTML %}
<div class="form-group">
    <label for="contents">표지이미지</label>
    <input  type="file" id="img_files" name="img_files" accept="image/*" class="form-control" onchange="__common.cmdSendByAjax(this.id, 'frm');">
    <input type="hidden" id="Key" name="Key" />
    <input type="hidden" id="Location" name="Location" />
</div>
{% endhighlight %}


## action을 수행하는 script파일.
{% highlight js %}
__common = {
    cmdSendByAjax : function (_obj, _form) {
    $('form').ajaxForm({
        url: '/util/FileUpload',
        enctype: 'multipart/form-data',

        beforeSubmit: function (data, form, option) {
            console.info('beforeSubmit');
            console.info(data);
            console.info(form);
            console.info('beforeSubmit');
            //validation체크
            //막기위해서는 return false를 잡아주면됨
            return true;
        },
        success: function (response, status) {
            console.info(response);
            $('#Location').val(response.Location);
        },
        error: function () {
            console.info('error');
        }
    });
    $('#' + _form).submit();
}
}
{% endhighlight %}


## router의 util.js
- aws기본 설정
{% highlight js %}
AWS.config.loadFromPath('./config/config.json');
AWS.config.region = 'ap-northeast-2'; //지역 설정
{% endhighlight %}

- AWS fileUpload 설정!!
{% highlight js %}
router.all('/FileUpload', function(req, res, next) {
var form = new multiparty.Form();

form.on('field',function(name,value){
    console.log('normal field / name = '+name+' , value = '+value);
});

//S3 버킷 설정
form.on('file',function(name, file){
    var param = {
        'Bucket':'BucketName', //s3에 설정한 버킷 이름 설정.
        'Key':file.originalFilename,
        'ACL':'public-read',
        'Body':null
    };

    var orgFileName = file.originalFilename;
    var orgFileExtension = orgFileName.lastIndexOf(".");
    
    //원본 파일명을 바꾸기 위한 코드.
    //파일명의 중복을 막기 위해 설정
    function makeid() {
        var text = "";
        var possible = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        
        for (var i = 0; i < 10; i++)
        text += possible.charAt(Math.floor(Math.random() * possible.length));
        
        return text;
    }

    param.Key = makeid() + orgFileExtension;

    //param.Key = file.originalFilename;
    param.Body = require('fs').createReadStream(file.path);
    
    //upload
    s3.upload(param, function(err, data){
        console.log(err);
        console.log(data);

        res.status(200).send(data);
    });
    
});

form.parse(req);
});
{% endhighlight %}


 `var param = {...}` 객체는 AWS S3업로드에 대한 정보
- Bucket : S3 Bucket 이름을 지정합니다.
- Key : S3의 경로 및 파일 이름을 지정합니다.
- ACL : 파일 권한에 대한 설정입니다.
- Body : 업로드할 파일의 경로를 지정합니다.



#### 출처
- [Node AWS S3 업로드/Yun Blog](https://cheese10yun.github.io/Node-AWS-S3-Upload/)