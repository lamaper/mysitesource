+++
date = '2023-08-20T12:00:00+08:00'
draft = true
title = 'Moectf2022的一些wp（已废弃）'
tags = ['ctf']
categories = ['ctf']
+++

MoeCTF2022 WP

@author：lamaper

@email：lamaper@qq.com

## web

### 1.ezhtml

浏览器按F12查看网络流，发现文件evil.js，双击打开

```javascript
var sx = document.querySelector('#sx');
var yw = document.querySelector('#yw');
var wy = document.querySelector('#wy');
var zh = document.querySelector('#zh');
var zf = document.querySelector('#zf');
var arr = [sx, yw, wy, zh];
var flag = false;
function check() {
    if (flag == true) {
        clearInterval(timer);
    }
    var sum = 0;
    for (var i = 0; i < arr.length; i++) {
        sum += eval(arr[i].innerHTML);
    }
    if (sum == eval(zf.innerHTML) && sum > 600) {
        alert('moectf{W3lc0me_to_theWorldOf_Web!}');
        flag = true;
    }
}
var timer = setInterval(check, 1000);
```

发现flag``moectf{W3lc0me_to_theWorldOf_Web!}``

### 2.God_of_Aim

浏览器F12查看网络流，发现三个js文件target.js、index.js、aimtrainer.js

在aimtrainer.js中发现代码被混淆成无法阅读的状态

```javascript
var _0x78bd=["\x61\x69\x6D\x54\x72\x61\x69\x6E\x65\x72\x45\x6C","\x61\x69\x6D\x2D\x74\x72\x61\x69\x6E\x65\x72","\x67\x65\x74\x45\x6C\x65\x6D\x65\x6E\x74\x42\x79\x49\x64","\x73\x63\x6F\x72\x65\x45\x6C","\x73\x63\x6F\x72\x65","\x61\x69\x6D\x73\x63\x6F\x72\x65","\x64\x65\x6C\x61\x79","\x74\x61\x72\x67\x65\x74\x53\x69\x7A\x65","\x61\x69\x6D\x73\x63\x6F\x72\x65\x45\x4C","\x73\x65\x74\x53\x63\x6F\x72\x65","\x73\x74\x61\x72\x74","\x69\x6E\x6E\x65\x72\x48\x54\x4D\x4C","\x73\x65\x74\x41\x69\x6D\x53\x63\x6F\x72\x65","\x70\x6F\x73\x69\x74\x69\x6F\x6E","\x73\x74\x79\x6C\x65","\x72\x65\x6C\x61\x74\x69\x76\x65","\x74\x69\x6D\x65\x72","\x63\x72\x65\x61\x74\x65\x54\x61\x72\x67\x65\x74","\x63\x68\x65\x63\x6B\x66\x6C\x61\x67\x31","\x63\x68\x65\x63\x6B\x66\x6C\x61\x67\x32","\x73\x74\x6F\x70","\x6D\x6F\x65\x63\x74\x66\x7B\x4F\x68\x5F\x79\x6F\x75\x5F\x63\x61\x6E\x5F\x61\x31\x6D\x5F","\u4F60\u5DF2\u7ECF\u5B66\u4F1A\u7784\u51C6\u4E86\uFF01\u8BD5\u8BD5\u770B\x3A","\x73\x74\x61\x72\x74\x32","\x61\x6E\x64\x5F\x48\x34\x63\x6B\x5F\x4A\x61\x76\x61\x73\x63\x72\x69\x70\x74\x7D",""];class AimTrainer{constructor({_0xf777x2,_0xf777x3}){this[_0x78bd[0]]= document[_0x78bd[2]](_0x78bd[1]);this[_0x78bd[3]]= document[_0x78bd[2]](_0x78bd[4]);this[_0x78bd[4]]= 0;this[_0x78bd[5]]= 0;this[_0x78bd[6]]= _0xf777x2|| 1000;this[_0x78bd[7]]= _0xf777x3|| 30;this[_0x78bd[8]]= document[_0x78bd[2]](_0x78bd[5])}createTarget(){const _0xf777x5= new Target({delay:this[_0x78bd[6]],targetSize:this[_0x78bd[7]],aimTrainerEl:this[_0x78bd[0]],onTargetHit:()=>{this[_0x78bd[9]](this[_0x78bd[4]]+ 1)}});_0xf777x5[_0x78bd[10]]()}setScore(_0xf777x7){this[_0x78bd[4]]= _0xf777x7;this[_0x78bd[3]][_0x78bd[11]]= this[_0x78bd[4]]}setAimScore(_0xf777x7){this[_0x78bd[5]]= _0xf777x7;this[_0x78bd[8]][_0x78bd[11]]= _0xf777x7}start1(){this[_0x78bd[9]](0);this[_0x78bd[12]](10);this[_0x78bd[0]][_0x78bd[14]][_0x78bd[13]]= _0x78bd[15];if(!this[_0x78bd[16]]){this[_0x78bd[16]]= setInterval(()=>{this[_0x78bd[17]]();this[_0x78bd[3]][_0x78bd[11]]= this[_0x78bd[4]];this[_0x78bd[18]]()},this[_0x78bd[6]])}else {return}}start2(){this[_0x78bd[7]]= 10;this[_0x78bd[6]]= 400;this[_0x78bd[9]](0);this[_0x78bd[12]](100000);this[_0x78bd[0]][_0x78bd[14]][_0x78bd[13]]= _0x78bd[15];if(!this[_0x78bd[16]]){this[_0x78bd[16]]= setInterval(()=>{this[_0x78bd[17]]();this[_0x78bd[3]][_0x78bd[11]]= this[_0x78bd[4]];this[_0x78bd[19]]()},this[_0x78bd[6]])}else {return}}checkflag1(){if(this[_0x78bd[4]]== this[_0x78bd[5]]){this[_0x78bd[20]]();alert(_0x78bd[21]);alert(_0x78bd[22]);this[_0x78bd[23]]()}}checkflag2(){if(this[_0x78bd[4]]== this[_0x78bd[5]]){this[_0x78bd[20]]();alert(_0x78bd[24])}}stop(){this[_0x78bd[0]][_0x78bd[11]]= _0x78bd[25];if(this[_0x78bd[16]]){clearInterval(this[_0x78bd[16]]);this[_0x78bd[16]]= 0}else {return}}}
```

利用反混淆工具[obfuscator解密 - dejs.vip (idd1.com)](http://www.idd1.com/encry_decry/obfuscator.html)进行反混淆，得到代码：

```javascript
class AimTrainer {
    constructor({_0xf777x2, _0xf777x3}) {
        this.aimTrainerEl = document.getElementById("aim-trainer");
        this.scoreEl = document.getElementById("score");
        this.score = 0;
        this.aimscore = 0;
        this.delay = _0xf777x2 || 1000;
        this.targetSize = _0xf777x3 || 30;
        this.aimscoreEL = document.getElementById("aimscore");
    }
    createTarget() {
        const target = new Target({
            delay: this.delay,
            targetSize: this.targetSize,
            aimTrainerEl: this.aimTrainerEl,
            onTargetHit: () => {
                this.setScore(this.score + 1);
            }
        });
        target.start();
    }
    setScore(scoreValue) {
        this.score = scoreValue;
        this.scoreEl.innerHTML = this.score;
    }
    setAimScore(scoreValue) {
        this.aimscore = scoreValue;
        this.aimscoreEL.innerHTML = scoreValue;
    }
    start1() {
        this.setScore(0);
        this.setAimScore(10);
        this.aimTrainerEl.style.position = "relative";
        if (!this.timer) {
            this.timer = setInterval(() => {
                this.createTarget();
                this.scoreEl.innerHTML = this.score;
                this.checkflag1();
            }, this.delay);
        } else {
            return;
        }
    }
    start2() {
        this.targetSize = 10;
        this.delay = 400;
        this.setScore(0);
        this.setAimScore(100000);
        this.aimTrainerEl.style.position = "relative";
        if (!this.timer) {
            this.timer = setInterval(() => {
                this.createTarget();
                this.scoreEl.innerHTML = this.score;
                this.checkflag2();
            }, this.delay);
        } else {
            return;
        }
    }
    checkflag1() {
        if (this.score == this.aimscore) {
            this.stop();
            alert("moectf{Oh_you_can_a1m_");
            alert("你已经学会瞄准了\uFF01试试看:");
            this.start2();
        }
    }
    checkflag2() {
        if (this.score == this.aimscore) {
            this.stop();
            alert("and_H4ck_Javascript}");
        }
    }
    stop() {
        this.aimTrainerEl.innerHTML = "";
        if (this.timer) {
            clearInterval(this.timer);
            this.timer = 0;
        } else {
            return;
        }
    }
}
```

得到flag``moectf{Oh_you_can_a1m_and_H4ck_Javascript}``

