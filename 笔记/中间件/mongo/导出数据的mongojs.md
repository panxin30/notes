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
# 打印mongo数据范例
```
var idd = ["4bd96ed4-92ff-467a-a30a-f7b55efd03c7","f280b0d7-cd94-42fa-b6c9-7e892eb50f5d","e33351ac-b76d-409b-8948-77f7366c1f2b","a1c77309-dfcb-4a0e-a0c4-4fe9d140fe42","6bd429e5-38ab-4c92-8c89-4ecd20da3e57","b8da6fe2-5335-4763-8561-ceb2b5b7bfc6","615a0653-d889-4b11-b8cc-fa39471322cd","2744c8a8-9371-458f-8de7-9c528e5a01da","773f4813-8bdd-4c46-ac5a-860766826820","a06660f2-2b9b-4f05-b76e-38614ed1b4b8","c1ae35f2-d896-471b-80d8-f4e52ca8fa88","b0029998-157f-4bb9-9a24-d46db20b2b1b","fe66fdee-db1f-47a5-87ba-7d7056985835","f7536eb1-246c-415a-9a81-525d0c5f7f57","de926ee3-dddf-4625-873a-81e87d60e705","d4c1ca72-56f1-40a6-a2d1-871509ab2e1a","868bd909-f665-4409-ad52-b1bf2711e4b7","025a12e7-4888-4a4f-8e58-1734160f4d1a","00bdf0b7-ec8a-405d-a366-df19a16d41ce","2c9564fd-d4fb-4830-9fad-a1032ba701f3","d1c1fb18-2180-4dcf-8cbe-d24723d7b4e7","0f04591b-b292-4555-bcb8-161b6fe8c962"]; 
 
 
 
db.getCollection("t_financial_info").find({"tenantId" : "T002561","customerId":{$in:idd}}).forEach(function(item){ 
 
    print("db.t_financial_info.insert(" + tojson(item) +");") 
 
}) 
```