# Ticketing-Tool-Order-Delivery-
function doGet(e) {
  var page = e.parameters.view != undefined ? e.parameters.view : "login";
  var password;
  if(page == "login"){
    var template = HtmlService.createTemplateFromFile(page);
    return template.evaluate();
  }else if(page == "user"){
    password = UTILITIES.getPassword(e.parameters.u);
  }else{
    password = UTILITIES.getPassword(page.toString().toUpperCase());
  }
  var ts = e.parameters.t;
  var cache = CacheService.getScriptCache();
  var cachedRequest = JSON.parse(cache.get("request"));
  var reqPassword = cachedRequest[page][ts];
  if(reqPassword == password){
    var template = HtmlService.createTemplateFromFile(page);
    return template.evaluate();
  }else{
    return HtmlService.createHtmlOutput("<html>incorect Password</html>");
  }
}

function callApi(type,dataObj,reqObj){
  var payload = UTILITIES.createPayload(type,dataObj,reqObj);
  var response = UTILITIES.postCall(payload,UTILITIES.getApi("REQUISITION"));
  return response;
}

function include(filename) {
  return HtmlService.createHtmlOutputFromFile(filename).getContent();
}

function getStaticData() {
  var contents = callApi('Static',{},{});
  return contents["message"]["Vendor"];
}

function renderView(page,password,user){
  var ts = new Date().valueOf();
  var cache = CacheService.getScriptCache();
  var cachedRequest = JSON.parse(cache.get("request"));
  if (cachedRequest == null) {
    cachedRequest = {};
  }
  if(cachedRequest[page] == undefined){
    cachedRequest[page] = {};
  }
  cachedRequest[page][ts]=password;
  cache.put("request", JSON.stringify(cachedRequest), 120);
  return (ScriptApp.getService().getUrl()).concat("?view=").concat(page).concat("&t=").concat(ts).concat("&u=").concat(user);
}

