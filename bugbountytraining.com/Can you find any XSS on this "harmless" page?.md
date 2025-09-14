# Can you find any XSS on this "harmless" page?
## target: https://www.bugbountytraining.com/challenges/challenge-8.php

# Rootcause
```
<script type="text/javascript"> 
function cfpParam(name) {
        var regex = new RegExp("[#]" + name + "=([^\\?&#]*)");
        var t = window.location.href;
        var loc=t.replace(/%23/g,"#");
        var results = regex.exec(loc);
        return (results === null) ? "" : unescape( results[1] );
}
function cfpMatchDef(val,regex,def) {
        var results = regex.exec(val);
        return (results === null) ? def : val;
}
function cfpAlphaParam(name,def) {
        var regex = new RegExp("^[a-zA-Z0-9.!?; \t_]+$");
        return cfpMatchDef(cfpParam(name),regex,def);
}
var cfpPid= cfpAlphaParam("pid",0);
var cfpPrBase="https://www.bugbountyhunter.com/";
var cfpClick = cfpParam("clk");
var cfpOrd = cfpParam("n");
if(cfpOrd === ""){
var axel = Math.random() + "";
cfpOrd = axel * 1000000000000000000;
}
 
function pr_swfver(){
var osf,osfd,i,axo=1,v=0,nv=navigator;
if(nv.plugins&&nv.mimeTypes.length){osf=nv.plugins["Shockwave Flash"];if(osf&&osf.description){osfd=osf.description;v=parseInt(osfd.substring(osfd.indexOf(".")-2))}}
else{try{for(i=5;axo!=null;i++){axo=new ActiveXObject("ShockwaveFlash.ShockwaveFlash."+i);v=i}}catch(e){}}
return v;
}
var pr_d=new Date();pr_d=pr_d.getDay()+"|"+pr_d.getHours()+":"+pr_d.getMinutes()+"|"+-pr_d.getTimezoneOffset()/60;
var pr_redir=cfpClick+"$CTURL$";
var pr_nua=navigator.userAgent.toLowerCase();
var pr_sec=((document.location.protocol=='https:')?'&secure=1':'');
var pr_pos="",pr_inif=(window!=top);
if(pr_inif){try{pr_pos=(typeof(parent.document)!="unknown")?(((typeof(inDapIF)!="undefined")&&(inDapIF))||(parent.document.domain==document.domain))?"&pos=s":"&pos=x":"&pos=x";}
catch(e){pr_pos="&pos=x";}if(pr_pos=="&pos=x"){var pr_u=new RegExp("[A-Za-z]+:[/][/][A-Za-z0-9.-]+");var pr_t=this.window.document.referrer;
var pr_m=pr_t.match(pr_u);if(pr_m!=null){pr_pos+="&dom="+pr_m[0];}}else{if(((typeof(inDapMgrIf)!="undefined")&&(inDapMgrIf))||((typeof(isAJAX)!="undefined")&&(isAJAX))){pr_pos+="&ajx=1"}}}
var pr_s=document.location.protocol+"//"+cfpPrBase+"&flash="+pr_swfver()+"&time="+pr_d+"&redir="+pr_redir+pr_pos+pr_sec+"&r="+cfpOrd;
document.write("<script src='"+pr_s+"'><\/script>");
</script>
```
Trong script có một điểm nguy hiểm chính: cuối script nó dùng document.write để chèn một thẻ <script src='...'>, và phần src đó được xây từ nhiều giá trị lấy trực tiếp từ URL fragment (hash) — nhất là clk (cfpClick) — mà không mã hóa/escape khi chèn vào chuỗi HTML.

Nếu tham số (ví dụ clk) chứa một dấu nháy đơn (') hoặc các ký tự đặc biệt khác, chuỗi pr_s sẽ chứa dấu ' và khi document.write("<script src='"+pr_s+"'>") thực hiện, dấu ' đó sẽ đóng sớm thuộc tính src và phần dữ liệu tiếp theo sẽ được coi là thuộc tính mới của thẻ <script> — ví dụ onerror=... — từ đó có thể chèn và thực thi JS → XSS (injection vào thuộc tính thẻ).
# Poc
```
cfpParam("clk") lấy giá trị sau #clk= trong URL fragment (không decode đầy đủ, dùng unescape cũ).

pr_redir = cfpClick + "$CTURL$" (nối clk vào).

pr_s = document.location.protocol + "//" + cfpPrBase + ... + "&redir=" + pr_redir + ... → đây là chuỗi URL cuối cùng.

document.write("<script src='"+pr_s+"'><\/script>"); — chèn một script tag bằng chuỗi HTML. Nếu pr_s chứa dấu ', chuỗi HTML sẽ bị phá cấu trúc và nội dung sau dấu ' trở thành các attribute/JS mà browser sẽ parse.

Kết quả: attacker-controlled phần trở thành attribute (ví dụ onerror) và có thể chứa mã JS thực thi.

Yếu tố giúp tấn công:

Không có encode/escape khi chèn vào attribute HTML.

cfpParam cho phép ký tự ' vì regex chỉ cắt tại ?&# mà không loại '.

document.write với chuỗi HTML thô dễ bị DOM injection nếu không sanitize.
```
<img width="1442" height="579" alt="image" src="https://github.com/user-attachments/assets/09d36fc8-8808-4c0d-ae19-6e5441212bb6" />
