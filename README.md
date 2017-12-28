let jwt = require('jwt-simple');
let secret = "wangyy";
let time = 10;
module.exports = {
  /**
   * 检验token合法性
   */
   validate:function(req,res,next){
        let token = req.body.token||req.headers["xssToken"];
        if(token){
           let decodeToken = null;
           try {
                //防止假冒token解析報錯
                decodeToken = jwt.decode(token,secret,'HS256');
           } catch (err) {
                res.status(401).send("非法访问");
                return;
           }
           let exp = decodeToken.exp;
           if(!exp){

              res.status(401).send("非法访问");
           }
           let now = new Date().getTime();
           if(exp>(now+time*60*1000)){
             res.send({code:'002',"errorMsg":"授权超时"})
           }
           next();
        }else{
           res.status(401).send("非法访问");
        }
   },
   /*
    *生成token
    */
   makeToken(){
     let Token = null;
     let payload = {
         time:new Date().getTime(),
         exp:this.makeExp(time)
     }
     Token = jwt.encode(payload,secret,HS256)
     return Token;
   },
   /** [description]
    *生成token过期时间
    */
   makeExp:function(time){
       let stam = time*60*1000;
       return new Date().getTime()+stam;
   }
}


let express = require("express");
let app = express();
let bodyParser = require('body-parser');
let auth = require('./lib/auth.js');
let chalk = require('chalk');
    app.use(bodyParser.json());
    app.post('/login',function(req,res,next){
             let Token = auth.makeToken();
             res.json({result:"success",token:Token},200)
    });
    app.use('*',[auth.validate],function(req,res,next){
       res.send('success');
    });
    app.listen('9999')

