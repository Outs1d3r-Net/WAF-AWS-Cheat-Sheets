# AWS WAF - Cheat Sheet  
[![Banner](img/waf.png#center)](waf)
#  
:metal: :smile:  
#

## Introdução  
Este guia foi desenvolvido para ser um guia rápido para iniciantes no WAF da AWS.  

Ele contem informações para você criar filtros e ter visibilidade do trafego da WEB que chega ate os seus sistemas que passam pelo WAF & Shield.  

Você encontrará um guia técnico do recurso [aqui.](https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html)  
#  
#### O WaF da AWS  
O WaF da AWS conta com algumas funções de pesquisa quando analisado pelo cloudwatch, acesse o Cloudwatch na AWS e selecione: ```Logs Insights``` no canto esquerdo:  
#  
[![Console Cloudwatch](img/00.png#center)]()  
#  

A tela do console de pesquisa do CloudWatch se abrirá:  
#  
[![Console Cloudwatch](img/01.png#center)]()  
#  

No CloudWatch, selecione a(s) trilha(s) que você deseja:  
#  
[![Trilhas do waf no cloudwatch](img/02.png#center)]()  
#  

## Operadores  
No WAF da AWS os operadores nos ajudaram a criar consultar mais afinadas para que possamos ter visibilidade dos dados que desejamos.  
#  
+ ### fields:  
O operador ```fields``` é utilizado para "setar" os campos que desejamos pesquisar, por exemplo, caso queria redirecionar suas pesquisas para o campo ```@message``` e ```@timestamp``` :  
```
fields @timestamp, @message
```  
O campo ```@message``` contém todos os dados de uma requisição, este campo esta no formato JSON;  

O campo ```@timestamp``` contém dados de data e hora da requisição;  
#  
+ ### filter:  
O operador ```filter``` é utilizado para realizar filtros mais afinados em um determinado campo, que foi definido com o operador ```fields```.  
```
fields @timestamp, @message
 | filter @message like /<STRING>/
```  
O operador ```filter``` também pode ser utilizado para selecionar a consulta em uma ```webACL``` especifica:  
```
fields @timestamp, @message
| filter webaclId = "arn:aws:wafv2:us-east-1:<anonymized>:regional/webacl/<anonymized>/<anonymized>-0f62-<anonymized>-8f8d-<anonymized>"
```  
#  
+ ### like:  
O operador ```like``` é utilizado para fazer match, por tanto utiliza-lo com o operador ``filter`` da a possibilidade de filtrar somente o que for definido para ``like``, para definir o valor de ``like`` atribua o valor entre slash's ```(/)``` :  
```
fields @timestamp, @message
 | filter @message like /STRING_HERE/
```  
#  
+ ### parse:  
O operador ```parse``` pode ser utilizado para analisar um campo definido de forma um pouco mais afinada do que o operador ```filter``` e ```like```.  
Dentro do campo ```@message``` temos varios dados:  
```  
Einstein@BlackHole:/tmp# cat out|jq
{
  "timestamp": 1651867777491,
  "formatVersion": 1,
  "webaclId": "arn:aws:wafv2:us-east-1:<anonymized>:regional/webacl/<anonymized>/<anonymized>",
  "terminatingRuleId": "<anonymized>",
  "terminatingRuleType": "REGULAR",
  "action": "ALLOW",
  "terminatingRuleMatchDetails": [],
  "httpSourceName": "ALB",
  "httpSourceId": "<anonymized>-app/lb-<anonymized>-production/<anonymized>",
  "ruleGroupList": [],
  "rateBasedRuleList": [],
  "nonTerminatingMatchingRules": [],
  "requestHeadersInserted": null,
  "responseCodeSent": null,
  "httpRequest": {
    "clientIp": "111.111.111.111",
    "country": "AR",
    "headers": [
      {
        "name": "Accept-Encoding",
        "value": "gzip, identity"
      },
      {
        "name": "Host",
        "value": "www.anonymized.com
      },
      {
        "name": "Connection",
        "value": "keep-alive"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "\" Not A;Brand\";v=\"99\", \"Chromium\";v=\"100\", \"Google Chrome\";v=\"100\""
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "<anonymized>"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "<anonymized>"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "00-87d8bbd3205026fef9a57637c1f0b22f-<anonymized>-01"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "?0"
      },
      {
        "name": "user-agent",
        "value": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.2.4453.127 Safari/511.11"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "<anonymized>"
      },
      {
        "name": "accept",
        "value": "application/json, text/javascript, */*; q=0.01"
      },
      {
        "name": "x-requested-with",
        "value": "XMLHttpRequest"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "\"Windows\""
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "same-origin"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "cors"
      },
      {
        "name": "sec-fetch-dest",
        "value": "empty"
      },
      {
        "name": "referer",
        "value": "https://www.anonymized.com/test"
      },
      {
        "name": "accept-language",
        "value": "es-ES,es;q=0.9"
      },
      {
        "name": "X-Forwarded-For",
        "value": "111.111.111.111"
      },
      {
        "name": "X-Forwarded-Proto",
        "value": "https"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "<anonymized>"
      },
      {
        "name": "<COOKIE_anonymized>",
        "value": "<anonymized>"
      }
    ],
    "uri": "/test.json",
    "args": "utf8=%E2%9C%93&fild=&invisible_captcha=&promotion_id=&act=&fac=&lat=19.3866186&long=-99.111111&page=2&promo=",
    "httpVersion": "HTTP/1.1",
    "httpMethod": "GET",
    "requestId": "1-<anonymized>-<anonymized>"
  }
}
```  
Voce pode encontrar campos diferentes em solicitações diferentes.  
O operador ```parse``` nos da a possibilidade de realizar pesquisa nesses campos:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
| display UA
| limit 10
```  
Aqui estamos realizando um ```parse``` no campo ```name``` que possui o valor ```User-agent``` que possui o valor de caractere curinga: ```*``` ou seja, todos os UA's.  
Podemos tambem realizar esta pesquisa definindo um UA de nossa preferencia:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
| filter UA like /Mozilla/
| filter UA like /Windows NT/
| filter UA like /100.2.4453.127/
| filter UA like /Safari/
| limit 10
```  
#  
+ ### display:  
O operador ```display``` é utilizado para exibir um campo ou uma variavel que definimos.  
Por exemplo, podemos definir variadeis com o operador ```as```:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
```  
Todos os resultados desse ```parse``` foram marcados como uma variavel que chamados de ```UA```.  

O operador ```display``` permite exibir somente este campo no resultado:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
| display UA
```  
#  
+ ### stats:  
O operador ```stats``` realiza uma analise statistica. Abaixo estamos definindo um ```count``` no campo ```requestCount``` das solicitações:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
| stats count(*) as requestCount by UA
```  
#  
+ ### sort:  
O operador ```sort``` é utilizado para realizar operações de ordenação, o mais comum é utilizar o operador ```desc``` em conjunto:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
| stats count(*) as requestCount by UA
| sort requestCount desc
```  
#  
+ ### limit:  
O operador ```limit``` é utilizado para controlar o limite de dados de saida, dessa forma conseguimos controlar a quantidade de resultados na visualização.  

A query abaixo retorna um ```Top 10``` de UA's ```(User-agent)``` que estão batendo no WAF:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
| stats count(*) as requestCount by UA
| sort requestCount desc
| limit 10
```  
Veja a imagem abaixo:  
# 
[![Console Cloudwatch](img/03.png#center)]()  
# 

Para obter uma visão mais detalhada, seria bom se pudéssemos adicionar o IP para diferenciar os UA's repetidos:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
| stats count(*) as requestCount by httpRequest.clientIP,UA
| sort requestCount desc
| limit 10
```  
Veja a imagem abaixo:  
#  
[![Console Cloudwatch](img/04.png#center)]()  
#  
OBS: Diminuimos o tempo de consulta para 5 minutos para obter uma melhor visualização.  

## Templates de consultas  
Nesta sessão vou disponibilizar alguns modelos de consultas, algumas que eu mesmo ja utilizei ou utilizo.  
Sempre procurarei atualiza-los com o tempo.  
#  

+ ### TOP 10 IP's:  
Podemos obter um TOP 10 dos IPs que estão acessando seu site com a query abaixo:  
```
"fields @timestamp, @message
| filter webaclId = ""arn:aws:wafv2:us-east-1:<anonymized>:regional/webacl/<anonymized>/<anonymized>-0f62-<anonymized>-8f8d-<anonymized>""
| parse @message '{""name"":""X-Forwarded-For"",""value"":""*""}' as IPfinal
| stats count(*) as requestCount by IPfinal
| sort requestCount desc 
| limit 10"
```  
Também conseguimos realizar isso com a variavel ```httpRequest.clientIp``` da seguinte forma:  
```
fields @timestamp, @message
| display httpRequest.clientIp
| stats count(*) as requestCount by httpRequest.clientIp
| sort requestCount desc 
| limit 10
```  
Veja a imagem abaixo:  
# 
[![Console Cloudwatch](img/05.png#center)]()  
#  
+ ### Encontrando STRINGS especificas:  
Utilizando combinações com o operador ```filter``` e ```like``` conseguimos buscar STRINGS especificas, isso é util quando temos alguns dados chaves de um incidente:  
```
fields @timestamp, @message
| sort @timestamp desc
| filter @message like /STRING_HERE/
| limit 100
```  
Veja a imagem abaixo:  
# 
[![Console Cloudwatch](img/06.png#center)]()  
OBS: No print acima eu adicionei os campos ```httpRequest.headers.1.value``` e ```httpRequest.clientIp``` para melhorar a visualização dos resultados.  
#  
* ### Obtenha dados da web de um determinado IP:  
Esta query é excelente para bisbilhotar os dados que estão sendo enviados para o seu servidor nas requisições web:  
```
fields @timestamp, @message
| sort @timestamp desc
| filter @message like /IP_ADDRESS_HERE/
| display httpRequest.clientIp, httpRequest.country, httpRequest.httpMethod, httpRequest.uri, httpRequest.args, httpRequest.httpVersion
| limit 100
```  
Veja a imagem abaixo:  
# 
[![Console Cloudwatch](img/07.png#center)]()  
# 
Vamos falar sobre as variaveis acima:  
+ #### httpRequest.clientIp:  
Esta retorna o endereço de IP de quem realizou a requisição;  
+ #### httpRequest.country:  
Esta retornará dados do país de origem da requisição;  
+ #### httpRequest.httpMethod:  
Aqui retornamos o methodo HTTP, util para refinar ainda mais suas pesquisas;  
+ #### httpRequest.uri:  
Aqui retornamos a URI da requisição, sabendo que a URL é um conjunto de Protocolo, IP, e Path, a URI seria a Path completa;  
+ #### httpRequest.args:  
Este nos retornará os argumentos da requisição, sabendo que os argumentos são o que geralmente passamos como entrada, ex: https://www.google.com/search?q=aws+waf  
+ Este é o Protocolo: ``` https://``` ;  
+ Esta é a URL: ```https://www.google.com/search?q=aws+waf``` ;  
+ Este é a URI: ```/search``` ;  
+ Este é o argumento: ```?q=aws+waf``` ;  
+ #### httpRequest.httpVersion:  
Este é a versão do protocolo que esta sendo invocado.  
#  
+ ### Encontrando UA's especificos:  
```
fields @timestamp, @message
| parse @message '{"name":"user-agent","value":"*"}' as UA
| filter UA like /Mozilla/
| filter UA like /Windows NT/
| filter UA like /100.2.4453.127/
| filter UA like /Safari/
| limit 10
```  

#### Estero que este pequeno guia ajude as pessoas a desenvolverem suas tecnicas de consulta no WAF da AWS (WAF & SHIELD) por favor fique a vontade para me ajudar a alimentar este repositorio, acredito que podemos aprender muitas coisas juntos.

Com grandes poderes, grandes responsabilidades...  

### Leitura adicional:    
[https://docs.aws.amazon.com/waf/latest/developerguide](https://docs.aws.amazon.com/waf/latest/developerguide)   
[https://aws.amazon.com/waf/features/](https://aws.amazon.com/waf/features/)   
[https://aws.amazon.com/waf/pricing/](https://aws.amazon.com/waf/pricing/)   
[https://aws.amazon.com/waf/faqs/](https://aws.amazon.com/waf/faqs/)   


