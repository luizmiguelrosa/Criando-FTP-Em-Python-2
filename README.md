<h1 align="center">Criando FTP Em Python 2</h1>
 
Esta é a continuação do [Criando FTP em Python](https://github.com/Masso13/Criando-FTP-Em-Python), mas com explicações avançadas para complementar o seu FTP.

## Ignorando o usuário "anonymous"
```python
class DummyMD5Authorizer(DummyAuthorizer):
    def validate_authentication(self, username, password, handler):
        if username != "anonymous":
            ....

authorizer = DummyMD5Authorizer()
```

## Fazendo Login com segurança
```python
from pyftpdlib.authorizers import DummyAuthorizer, AuthenticationFailed

class DummyMD5Authorizer(DummyAuthorizer):
    def validate_authentication(self, username, password, handler):
        if username != "anonymous":
            password = password.encode("latin1")
            hash = md5(password).hexdigest()
            try:
                if user_password != hash:
                    raise KeyError
            except KeyError:
                raise AuthenticationFailed()

authorizer = DummyMD5Authorizer()
```
A variável `user_password` é um exemplo, mas você pode colocar alguma consulta em um banco de dados ou de um dicionário.

## Limitando a taxa de banda
```python
from pyftpdlib.handlers import FTPHandler, ThrottledDTPHandler

dtp_handler = ThrottledDTPHandler
dtp_handler.read_limit = 30720
dtp_handler.write_limit = 30720

handler = FTPHandler
handler.dtp_handler = dtp_handler
```
Os parâmetros `read_limit` e `write_limit` são para determinar a banda máxima para cada usuário. Os valores são em **Bytes**.

**KB:** valor * 1024<br>
**MB:** valor * 1000000

## Transferencia rápida
```python
handler = FTPHandler
handler.use_sendfile = True
```
Isto funciona apenas em sistemas Unix. Mais facilmente dizendo, em Linux. Então recomendo para aqueles que queiram criar um servidor fixo para altas transferências.

## Usando os usuários do sistema (Linux)
```python
from pyftpdlib.authorizers import UnixAuthorizer

authorizer = UnixAuthorizer(rejectec_users=["root"], require_valid_shell=False)

# Modificar as informações do usuário dentro do FTP
authorizer.override_user("usuario", password="senha", perm="elr")
```
Para usar os usuários do Windows, use `WindowsAuthorizer`.