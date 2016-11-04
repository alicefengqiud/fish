var aneObj = function() {
    //x轴
    this.x = [];
    //高度
    this.len = [];

};
aneObj.prototype.num = 50;

aneObj.prototype.init = function(){
    for(var i=0;i<this.num;i++){
        this.x[i] = i * 16 + Math.random() * 20;
        this.len[i] = 200 + Math.random() * 50;
    }
};
//绘制海葵
aneObj.prototype.draw = function(){

    ctx2.save();
    ctx2.globalAlpha = 0.6;
    ctx2.lineWidth = 20;
    ctx2.lineCap = "round";
    ctx2.strokeStyle = "#3e154e";
    for(var i=0;i < this.num;i++){
        //beginPath,moveTo,lineTo,stroke,strokeStyle,lineWidth,lineCap,globalAlpha
        ctx2.beginPath();
        ctx2.moveTo(this.x[i],canHeight);
        ctx2.lineTo(this.x[i],canHeight-this.len[i]);
        ctx2.stroke();
    }
    ctx2.restore();  //一旦出去，样式就没有了
};
