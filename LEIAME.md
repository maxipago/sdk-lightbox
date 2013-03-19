## INTRODUÇÃO ##

A **maxiPago!** oferece aos seus clientes e desenvolvedores uma maneira inovadora para integrar o seu carrinho com a nossa plataforma de pagamentos: a biblioteca **Lightbox**.

Quando chamada, a lib _**maxiPago.js**_ abre um modal mostrando o formulário de pagamento pronto para receber os dados do comprador. Ela envia o pagamento direto para os servidores da **maxiPago!**, sem que nenhuma informação do cartão seja manuseada pelo Lojista.

Ela permite uma integração PCI que não requer que o comprador saia do seu site.

![maxiPago! Lightbox](https://raw.github.com/maxipago/Lightbox-Integration-lib/master/test/lightbox.png)

## INSTALAÇÃO ##

A Lightbox contém os seguintes arquivos:

```
  /lib/  
  |-- maxiPago-1.1.js  
  |-- jquery-1.7.2.min.js   
  /maxipago/  
    |-- maxipago.css  
    |-- config.xml  
    |-- dictionary.xml
    |-- Return.htm  
```

Insira o JavaScript da **maxiPago!** e o **jQuery 1.7.2+** em seu código, logo antes da tag **&lt;/body&gt;** para que a página carregue mais rapidamente:

```html
...
<script src="/caminho/do/diretorio/jquery-1.7.2.min.js"></script>
<script src="/caminho/do/diretorio/maxiPago-1.1.js"></script>
</body>
```

Note que a estrutura de diretórios da biblioteca deve permanecer intacta. A **maxiPago.js** sempre buscará seus arquivos de configuração dentro de /maxipago/ no mesmo diretório onde está instalada.

O arquivo **Return.htm** também deve ser colocado em seu servidor, mas em qualquer diretório de sua escolha. Ela é a página de Postback que receberá a resposta da transação.

Esta á uma solução *client-side*, logo todos os arquivos devem estar em diretórios publicados.


## CONFIGURAÇÕES ##

As configurações da Lightbox são feitas no arquivo **config.xml**, e são carregadas toda a vez que a página é carregada. Isto quer dizer que é possível fazer alterações sem a necessidade de reiniciar serviços ou servidores.

* **merchantId:** ID de Loja criado pela maxiPago!
* **server_url:** URL de destino da transação, informada pela maxiPago!
* **server\_url\_response:** URL de resposta da transação. Deve apontar para o endereço do arquivo 'Return.htm' desta biblioteca
* **form_width:** Largura do modal em pixels
* **form_height:** Altura do modal em pixels
* **showRefNum:** Booleano que define se o Número do Pedido ( _**hp\_refnum**_ ) será mostrado no modal
* **showAmount:** Booleano que define se o Valor do Pedido ( _**hp\_amount**_) será mostrado no modal


## CSS E LABELS ##

A Lightbox da **maxiPago!** permite a customização total do seu look-and-feel e do texto contido no modal.

* **maxipago.css**: Contém todas as classes de CSS utilizadas pela Lightbox para mudar a disposição dos elementos do modal
* **dictionary.xml**: Contém todos os textos impressos na tela da Lightbox

Os detalhes de cada classe e texto podem ser encontrados nos comentários dos respectivos arquivos.


## REQUISIÇÃO ##

```
Por favor notifique nossa equipe de Suporte caso tenha a intenção de usar a Lightbox para a  
integração, pois há configurações que precisam ser feitas antes de permitir uma chamada aprovada
```
A requisição para a Lightbox é feita passando um _**struct**_ pelo método **$MP.Buy()**, da seguinte forma:

```js
	var transaction = {
		hp_processor_id: "1",
		hp_txntype: "sale",
		hp_method: "ccard",
		hp_amount: "10.00",
		hp_refnum: "ORD1028307193",
		hp_sig_timestamp: "1312201855",
		hp_sig_id: "1234",
		hp_sig_itemid: "1028307193",
		hp_signature: "0c067be27be8e6cc092b7fe23bdb9b77"
	};
	$MP.Buy(transaction);
```


### Campos obrigatórios ###

* **hp\_processor\_id**: Código da Adquirente que irá processar a transação. Use **1** para testar
* **hp_method**: Sempre **ccard**
* **hp_txntype**: Autorização ( **auth** ) ou Venda Direta ( **sale** )
* **hp_amount**: Valor da transação com decimais (10.00)
* **hp_refnum**: Identificador do pedido no Estabelecimento
* **hp\_sig\_timestamp**: Data/hora da transação em formato epoch, usado na assinatura de validação
* **hp\_sig\_id**: Número aleatório de **4 dígitos** usado na assinatura de validação
* **hp\_sig\_itemid**: Código do pedido, usado na assinatura de validação. Não deve ser igual a _**hp_refnum**_
* **hp_signature**: Assinatura HMAC-MD5 (detalhes abaixo)

### Assinatura em hash ###

Para garantir a segurança da transação e a integridade dos dados postados a **maxiPago!** utiliza uma assinatura única por pedido, que valida se os dados foram alterados por alguém.

A assinatura é um hash HMAC-MD5 baseados nos dados do pedido e uma chave secreta que só a **maxiPago!** e o Lojista sabem.

Os dados do pedido devem estar concatenados em uma ordem específica e separados por um **asterísco**. Para mais informações veja a [Implementação HMAC](http://github.com/maxipago/Hmac-Implementation).


### Campos opcionais ###

* **hp_currency**: Código da moeda, ISO 4217
* **hp\_number\_of\_installments**: Número de parcelas do pedido. Se a transação for à vista, **não enviar**
* **hp_baddr**: Endereço de cobrança 1 
* **hp_baddr2**: Endereço de cobrança 2
* **hp_bcity**: Cidade de cobrança
* **hp_bstate**: Estado de cobrança, 2 letras
* **hp_bzip**: CEP de cobrança
* **hp_bcountry**: País  de cobrança, 2 letras (ISO 3166-2)
* **hp_phone**: Telefone do cliente
* **hp_email**: E-mail do cliente

### Salvando um cartão automaticamente ###

A **maxiPago!** lhe permite salvar o número de cartão de crédito para usá-lo em futuras compras, retornando um token para armazenamento. Para fazê-lo os seguintes campos são obrigatórios:

* **hp\_save\_customer**: Booleano para criar novo perfil (1 ou 0)
* **hp_savepayment**: Booleano para salvar cartão sob o novo perfil (1 ou 0)
* **hp\_c\_customer_id**: ID do perfil do cliente (deve ser único)
* **hp\_c\_addr**: Endereço de cobrança do cartão 1
* **hp\_c\_addr2**: Endereço de cobrança do cartão 2
* **hp\_c\_city**: Cidade de cobrança do cartão
* **hp\_c\_state**: Estado de cobrança do cartão, 2 letras
* **hp\_c\_zip**: CEP de cobrança do cartão
* **hp\_c\_country**: País de cobrança do cartão, 2 letras (ISO 3166-2)
* **hp\_c\_phone**: Telefone de cobrança do cartão
* **hp\_c\_email**: E-mail de cobrança do cartão



## RESPOSTA ##

Quando a Lightbox recebe uma resposta da **maxiPago!** ela mostra uma mensagem de Sucesso ou Falha. Ao fechar o modal a biblioteca chama o método **respMPLib()**.

O método **respMPLib()** deve estar no seu carrinho e você deve usá-lo para tratar a resposta da transação de acordo com seu fluxo de pagamento. O objecto **$MP.resp** contém os campos da resposta e você obtém os seus values chamando **$MP.resp.nomedocampo**:

```js
function respMPLib() {
	if ($MP.resp.hp_responsecode == "0") { alert("Seu pagamento foi Aprovado"); }
	else { alert("Desculpe, seu pagamento foi negado"); }
}
```

### Campos da resposta ###

* **hp_responsecode**: Código de resposta da transação. **Aprovada = 0**
* **hp_responsemsg**: Mensagem de erro, se houver
* **hp_time**: Data e hora da transação
* **hp_refnum**: Confirmação do valor enviado na requisição
* **hp_transid**: Transaction ID criado pela **maxiPago!**. **Guarde-o para uso futuro**
* **hp_orderid**: Order ID criado pela **maxiPago!**. **Guarde-o para uso futuro**
* **hp_authcode**: Código de autorização da transação
* **hp_amount**: Confirmação do valor enviado na requisição
* **hp_processortxnid**: ID da transação da Adquirente
* **hp_processorrefno**: Número de Referência da Adquirente
* **hp\_fraud\_score**: Score de análise antifraude, se houver
* **hp\_customer\_token**: Customer ID criado pela **maxiPago!** quando salva um cartão automaticamente
* **hp\_payment\_token**: Token do cartão criado pela **maxiPago!** quando salva um cartão automaticamente
* **hp\_signature\_response**: Assinatura hash da resposta

## SEGURANÇA E COMPLIANCE ##

Esta Lightbox foi desenhada para ser *PCI compliant*, ou seja, ela não armazena nem loga os informações de cartão de crédito de forma alguma. Isto também evita que você, lojista, tenha acesso aos dados do cartão.

A **maxiPago.js** criptografa os dados da transação antes de postá-los à **maxiPago!**, porém, para ser *PCI compliant*, a Lightbox **precisa estar dentro de um ambiente seguro (HTTPS)**.

## SUPORTE ##

Fique a vontade para contactar nossa equipe de Suporte caso tenha alguma dúvida sobre a integração Lightbox da **maxiPago!** ou sobre nossa plataforma de pagamento como um todo. Você pode entrar em contato através do email suporte@maxipago.com.

## LICENÇA ##
Licenciado sobre a Licença Apache, Versão 2.0 (a "Licença"); você não poderá usar este arquivo exceto se cumprir a Licença. Você pode obter uma cópia em inglês da Licença em:
  
http://www.apache.org/licenses/LICENSE-2.0  
  
Exceto se requerido pela lei or em acordo por escrito, o software distribuído sob a Licença é distribuído COMO ESTÁ, SEM GARANTIA OU CONDIÇÕES ALGUMA, sejam expressas ou implícitas. Veja a Licença para o idioma específico que governa permissões e limitações sob a Licença.