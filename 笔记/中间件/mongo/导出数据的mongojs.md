用具体的租户id代替tony
```
var tenantId="tony";
var customerIds=[];
var customerMap=[];
var certMap=[];
var baseMap=[];

//获取客户信息map
db.t_customer_profiles.find({"tenantId":tenantId}).forEach(
    function(customer){
        customerIds.push(customer._id);
        customerMap[customer._id]=customer;
    }
);
//获取证件信息map
db.t_certificates_info.find({"customerId":{$in:customerIds}}).forEach(
    function(cert){
        certMap[cert.customerId]=cert;
    }
);
//获取基本信息map
db.t_account_base_info.find({"customerId":{$in:customerIds}}).forEach(
    function(base){
        baseMap[base.customerId]=base;
    }
);

//遍历客户，打印数据
for(var n=0;n<customerIds.length;n++){

    var cust=customerMap[customerIds[n]];//客户
    var cert=certMap[customerIds[n]];//证件
    var base=baseMap[customerIds[n]];//基本信息
    
   
    var account="";
    var serverId="";
    
    //账号组装成|分割的字符串
    if(cert!=null&&cert.accounts!=null&&cert.accounts.length>0){
        for(var i=0;i<cert.accounts.length;i++){
            account = account+cert.accounts[i].account+"|";
            serverId = cert.accounts[i].serverId;
        }
    }
     
    //获取各个字段，注意判空
     var idNum=(cert!=null&&cert.idNum!=undefined)?cert.idNum:"";
     var idType=(cert!=null&&cert.idType!=undefined)?cert.idType:"";
     var bankAccount =(cert!=null&&cert.bankAccount !=undefined)?cert.bankAccount:"";
     var bankBranch =(cert!=null&&cert.bankBranch !=undefined)?cert.bankBranch:"";
     var accountNo =(cert!=null&&cert.accountNo !=undefined)?cert.accountNo:"";
    
     var nationality =(base!=null&&base.nationality!=undefined)?base.nationality:"";
     var homePlace =(base!=null&&base.homePlace!=undefined)?base.homePlace:"";
     var postcode =(base!=null&&base.postcode!=undefined)?base.postcode:"";
     var address =(base!=null&&base.address!=undefined)?base.address:"";
    
     var email =(cust!=null&&cust.email!=undefined)?cust.email:"";
     var oweId =(cust!=null&&cust.oweId!=undefined)?cust.oweId:"";
     var oweName =(cust!=null&&cust.oweName!=undefined)?cust.oweName:"";
     var phones =(cust!=null&&cust.phones!=null)?cust.phones.phone:"";
       
    
    //打印数据
     print(
        cust._id +"," +
        cust.customName + "," +
        cust.tenantId + "," +
        email + "," +
        phones + "," +
        cust.customNo + "," +
        oweId + "," +
        oweName + "," + 
        cust.customerState + "," + 
        account + "," + 
        nationality + "," + 
        homePlace + "," + 
        postcode +"," + 
        address + "," + 
        idType + ",'" + 
        idNum + ",'" + 
        accountNo + "," + 
        bankAccount + "," + 
        bankBranch); 
}
```
```
var tenantId="T001182";
var customerIds=[];
var customerMap=[];
var certMap=[];
var baseMap=[];

//获取客户信息map
db.t_customer_profiles.find({"tenantId":tenantId}).forEach(
    function(customer){
        customerIds.push(customer._id);
        customerMap[customer._id]=customer;
    }
);
//获取证件信息map
db.t_certificates_info.find({"customerId":{$in:customerIds}}).forEach(
    function(cert){
        certMap[cert.customerId]=cert;
    }
);
//获取基本信息map
db.t_account_base_info.find({"customerId":{$in:customerIds}}).forEach(
    function(base){
        baseMap[base.customerId]=base;
    }
);

//遍历客户，打印数据
for(var n=0;n<customerIds.length;n++){

    var cust=customerMap[customerIds[n]];//客户
    var cert=certMap[customerIds[n]];//证件
    var base=baseMap[customerIds[n]];//基本信息
    
   
    var account="";
    var serverId="";
    
    //账号组装成|分割的字符串
    if(cert!=null&&cert.accounts!=null&&cert.accounts.length>0){
        for(var i=0;i<cert.accounts.length;i++){
            account = account+cert.accounts[i].account+"|";
            serverId = cert.accounts[i].serverId;
        }
    }
     
    //获取各个字段，注意判空
     var idNum=(cert!=null&&cert.idNum!=undefined)?cert.idNum:"";
     var idType=(cert!=null&&cert.idType!=undefined)?cert.idType:"";
     var bankAccount =(cert!=null&&cert.bankAccount !=undefined)?cert.bankAccount:"";
     var bankBranch =(cert!=null&&cert.bankBranch !=undefined)?cert.bankBranch:"";
     var accountNo =(cert!=null&&cert.accountNo !=undefined)?cert.accountNo:"";
    
     var nationality =(base!=null&&base.nationality!=undefined)?base.nationality:"";
     var homePlace =(base!=null&&base.homePlace!=undefined)?base.homePlace:"";
     var postcode =(base!=null&&base.postcode!=undefined)?base.postcode:"";
     var address =(base!=null&&base.address!=undefined)?base.address:"";
    
     var email =(cust!=null&&cust.email!=undefined)?cust.email:"";
     var oweId =(cust!=null&&cust.oweId!=undefined)?cust.oweId:"";
     var oweName =(cust!=null&&cust.oweName!=undefined)?cust.oweName:"";
     var phones =(cust!=null&&cust.phones!=null)?cust.phones.phone:"";
     var createTime = (cust!=null&&cust.createTime!=null)?cust.createTime:"";
       
    
    //打印数据
     print(
        cust._id +"," +
        cust.customName + "," +
        cust.tenantId + "," +
        email + "," +
        phones + "," +
        cust.customNo + "," +
        oweId + "," +
        oweName + "," + 
        cust.customerState + "," + 
        account + "," + 
        nationality + "," + 
        createTime + "," + 
        postcode +"," + 
        address + "," + 
        idType + ",'" + 
        idNum + "," + 
        accountNo + ",'" + 
        bankAccount + "," + 
        bankBranch + "," +
		tenantId); 
}
```