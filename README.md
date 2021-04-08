### Usando Maven por trás de um proxy


# 1. Introdução
Neste tutorial, vamos configurar o Maven para funcionar atrás de um proxy - uma situação comum em ambientes onde não nos conectamos diretamente à Internet.

Em nosso exemplo, nosso proxy está sendo executado na máquina ```proxy.isaccanedo.com``` e escuta as solicitações de proxy via HTTP na ```porta:80```. Também usaremos alguns sites internos em internal.isaccanedo.com onde não precisamos passar por um proxy.

# 2. Configuração de proxy
Primeiro, vamos definir uma configuração básica de proxy sem nenhuma credencial.

Vamos editar nosso Maven settings.xml geralmente encontrado em nosso diretório ```<user_home>/.m2```. Se ainda não houver um, podemos copiá-lo das configurações globais no diretório ```<maven_home>/conf```.

E agora vamos criar uma entrada ```<proxy>``` dentro da seção ```<proxies>```:

```
<proxies>
   <proxy>
        <host>proxy.isaccanedo.com</host>
        <port>80</port>
    </proxy>
</proxies>
```

Como também usamos um site local que não precisa passar pelo proxy, vamos especificá-lo em ```<nonProxyHosts>``` usando um ``` | ``` lista separada com nosso localhost:

```
<nonProxyHosts>internal.isaccanedo.com|localhost|127.*|[::1]</nonProxyHosts>
```

# 3. Adicionando credenciais
Se nosso proxy não estivesse protegido, isso é tudo de que precisaríamos; no entanto, o nosso é, então vamos adicionar nossas credenciais à definição de proxy:

```
<username>isaccanedo</username>
<password>changeme</password>
```

Não adicionamos entradas de nome de usuário/senha se não precisarmos delas - mesmo vazias - já que tê-las presentes quando o proxy não as deseja pode fazer com que nossos pedidos sejam negados.

Nossa configuração autenticada mínima agora deve ser assim:

```
<proxies>
   <proxy>
        <host>proxy.isaccanedo.com</host>
        <port>80</port>
        <username>isaccanedo</username>
        <password>changeme</password>
        <nonProxyHosts>internal.isaccanedo.com|localhost|127.*|[::1]</nonProxyHosts>
    </proxy>
</proxies>
```

Agora, quando executarmos o comando mvn, passaremos pelo proxy para nos conectar aos sites que procuramos.

### 3.1. Entradas opcionais
Vamos dar a ele o id opcional de ```isaccanedoProxy_Authenticated``` para torná-lo mais fácil de referenciar, no caso de precisarmos trocar de proxies:

```
<id>isaccanedoProxy_Authenticated</id>
```

Agora, se tivermos outro proxy, podemos adicionar outra definição de proxy, mas apenas um pode estar ativo. Por padrão, o Maven usará a primeira definição de proxy ativa que encontrar.

As definições de proxy estão ativas por padrão e obtêm a definição implícita:

```
<active>true</active>
```

Se quiséssemos tornar outro proxy o ativo, desativaríamos nossa entrada original definindo ```<active>``` como ```false```:

```
<active>false</active>
```

O valor padrão do Maven para o protocolo do proxy é HTTP, que é adequado para a maioria dos casos. Se nosso proxy usar um protocolo diferente, nós o declararemos aqui e substituiremos http pelo protocolo de que nosso proxy precisa:

```
<protocol>http</protocol>
```

Observe que este é o protocolo que o proxy usa - o protocolo de nossas solicitações (ftp://, http://, https://) é independente disso.

E aqui está a aparência de nossa definição de proxy expandida, incluindo os elementos opcionais:

```
<proxies>
   <proxy>
        <id>isaccanedoProxy_Authenticated</id>
        <active>true</active>
        <protocol>http</protocol>
        <host>proxy.isaccanedo.com</host>
        <port>80</port>
        <username>isaccanedo</username>
        <password>changeme</password>
        <nonProxyHosts>internal.isaccanedo.com|localhost|127.*|[::1]</nonProxyHosts>
    </proxy>
</proxies>
```

Então, é isso para nossa entrada de proxy básica, mas é seguro o suficiente para nós?

# 4. Protegendo nossa configuração

Agora, digamos que um de nossos colegas deseja que enviemos nossa configuração de proxy.

Não estamos muito interessados em enviar nossa senha em texto simples, então vamos ver como o Maven facilita a criptografia de nossas senhas.

### 4.1. Criação de uma senha mestra
Primeiro, vamos escolher uma senha mestra, diga ```IsAcCaNeDo123456```.

Agora vamos criptografar nossa senha mestre, inserindo-a no prompt ao executar:

```
mvn --encrypt-master-password
Master Password:
```

Depois de pressionar Enter, vemos nossa senha criptografada entre chaves:

`` 
{QFMlh/6WjF8H9po9UDo0Nv18e527jqWb6mUgIB798n4=}
```

### 4.2. Solução de problemas de geração de senha
Se virmos {} em vez do prompt Master Password: (isso pode acontecer ao usar o bash), precisaremos especificar a senha na linha de comando.

Vamos colocar a senha entre aspas para garantir que quaisquer caracteres especiais como ``` ! ``` não tem efeitos indesejáveis.

Então, vamos usar aspas simples se estivermos usando bash:

```
mvn --encrypt-master-password 'IsAcCaNeDo123456'
```

Ou use aspas duplas se estiver usando um prompt de comando do Windows:

```
mvn --encrypt-master-password "IsAcCaNeDo123456"
```

Agora, às vezes, nossa senha mestre gerada contém chaves, como neste exemplo com uma chave de fechamento, ```}```, após o ```UD```:

```
{QFMlh/6WjF8H9po9UD}0Nv18e527jqWb6mUgIB798n4=}
```

Nesse caso, podemos:

- execute o comando ```mvn –encrypt-master-password``` novamente para gerar outro (espero que sem uma chave);
- escape das chaves em nossa senha adicionando uma barra invertida na frente de ```{``` ou ```}```.

### 4.3. Criação de um arquivo ```settings-security.xml```
Agora vamos colocar nossa senha criptografada, com um ```\}``` de escape, em um arquivo chamado settings-security.xml em nosso diretório .m2:

```
<settingsSecurity>
    <master>{QFMlh/6WjF8H9po9UD\}0Nv18e527jqWb6mUgIB798n4=}</master>
</settingsSecurity>
```

Por último, o Maven permite adicionar um comentário dentro do elemento mestre.

Vamos adicionar algum texto antes do delimitador ``` { ``` da senha, tomando cuidado para não usar um {ou} em nosso comentário, pois o Maven os usa para encontrar nossa senha:

```
<master>Nós escapamos da chave com '\' {QFMlh/6WjF8H9po9UD\}0Nv18e527jqWb6mUgIB798n4=}</master>
```

### 4.4. Usando uma unidade removível
Digamos que precisamos ser extremamente seguros e queremos armazenar nossa senha mestre em um dispositivo separado.

Primeiro, colocaremos nosso arquivo settings-security.xml em um diretório de configuração em uma unidade removível, "R:":

```
R:\config\settings-security.xml
```

E agora, vamos atualizar o arquivo settings-security.xml em nosso diretório .m2 para redirecionar o Maven para nosso settings-security.xml real em nossa unidade removível:

```
<settingsSecurity>
    <relocation>R:\config\settings-security.xml</relocation>
</settingsSecurity>
```

O Maven agora lerá nossa senha mestre criptografada do arquivo que especificamos no elemento relocation, em nossa unidade removível.

# 5. Criptografar senhas de proxy

Agora que temos uma senha mestra criptografada, podemos criptografar nossa senha de proxy.

Vamos executar o seguinte comando e inserir nossa senha, "changeme", no prompt:

```
mvn --encrypt-password
Password:
```

Nossa senha criptografada é exibida:

```
{U2iMf+7aJXQHRquuQq6MX+n7GOeh97zB9/4e7kkEQYs=}
```

Nossa etapa final é editar a seção de proxy em nosso arquivo settings.xml e inserir nossa senha criptografada:

```
<proxies>
   <proxy>
        <id>isaccanedoProxy_Encrypted</id>
        <host>proxy.isaccanedo.com</host>
        <port>80</port>
        <username>isaccanedo</username>
        <password>{U2iMf+7aJXQHRquuQq6MX+n7GOeh97zB9/4e7kkEQYs=}</password>
    </proxy>
</proxies>
```

Salve isso, e agora o Maven deve ser capaz de se conectar à Internet por meio de nosso proxy, usando nossas senhas criptografadas.

# 6. Usando Propriedades do Sistema
Embora configurar o Maven por meio do arquivo de configurações seja a abordagem recomendada, podemos declarar nossa configuração de proxy por meio das Propriedades do sistema Java.

Se nosso sistema operacional já tiver um proxy configurado, poderíamos definir:

```
-Djava.net.useSystemProxies=true
```

Alternativamente, para que esteja sempre habilitado, se tivermos direitos de administrador, podemos definir isso em nosso arquivo ```<JRE>``` /lib/net.properties.

No entanto, vamos observar que embora o próprio Maven possa respeitar essa configuração, nem todos os plug-ins o fazem, portanto, ainda podemos obter conexões com falha usando este método.

Mesmo quando ativado, podemos substituí-lo definindo os detalhes do nosso proxy HTTP na propriedade do sistema http.proxyHost:

```
-Dhttp.proxyHost=proxy.isaccanedo.com
```

Nosso proxy está escutando na porta padrão 80, mas se escutasse na porta 8080, configuraríamos a propriedade http.proxyPort:

```
-Dhttp.proxyPort=8080
```

E para nossos sites que não precisam do proxy:

```
-Dhttp.nonLocalHosts="internal.baeldung.com|localhost|127.*|[::1]"
```

Portanto, se nosso proxy estiver em 10.10.0.100, podemos usar:

```
mvn compile -Dhttp.proxyHost=10.10.0.100 -Dhttp.proxyPort=8080 
-Dhttp.nonProxyHosts=localhost|127.0.0.1
```

Claro, se nosso proxy requer autenticação, também adicionaremos:

```
-Dhttp.proxyUser=baeldung
-Dhttp.proxyPassword=changeme
```

E se quisermos que algumas dessas configurações se apliquem a todas as nossas invocações Maven, podemos defini-las na variável de ambiente MAVEN_OPTS:

```
set MAVEN_OPTS= -Dhttp.proxyHost=10.10.0.100 -Dhttp.proxyPort=8080
```

Agora, sempre que executarmos ```mvn```, essas configurações serão aplicadas automaticamente - até sairmos.

# 7. Conclusão
Neste artigo, configuramos um proxy Maven com e sem credenciais e criptografamos nossa senha. Vimos como armazenar nossa senha mestre em uma unidade externa e também vimos como configurar o proxy usando as propriedades do sistema.

Agora podemos compartilhar nosso arquivo settings.xml com nossos colegas sem fornecer nossas senhas em texto simples e mostrar a eles como criptografar as suas!