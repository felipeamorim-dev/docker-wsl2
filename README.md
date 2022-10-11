# Tutorial de instalação do docker no wls 2 com Ubuntu 22.04

O processo de instalação do docker engine (nativo em sistemas linux) pode não ser muito familizar para os usuários de windows. E por esse motivo pode ocorre alguns empecílios. 

Esse tutorial é para resolver alguns problemas que podem ocorrem na instalação do docker no ubuntu 22.04 para o wls 2 em computadores diversos, sobretudo, em computadores com restrinções de segurança utilizados muitas vezes nos ambientes corporativos.

**Vamos iniciar o tutorial**

Primeiramente, vamos relembrar como realizar a instalação do wls de forma breve.

&nbsp;

## Instalação do wls

Realize a instalação do windows terminal pela loja de apps do windows. Certifique-se que está utilizando o windows 10 ou 11 atualizados em suas versões mais recentes. 

![Windows Terminal](https://github.com/felipeamorim-dev/docker-wls2/blob/main/assets/windows-terminal.png)

Execute o comando abaixo, no powershell em modo administrador:

```
wsl --install
```

Caso deseje realizar a instalação a partir de uma imagem de uma distribuição linux diferente, utilize o comando abaixo:

```
wsl --install -d <Nome da Distribuição>
```

Reinicie seu computador caso seja necessário.

Após baixar o Ubuntu 22.04.01 LTS da loja do windows, execute o instalador da distribuição e siga o passos indicados pelo instalador. Finalizada sua instalação do ubuntu, feche a janela e inicie o ubuntu pelo windows terminal.

Em uma nova guia do windows terminal com o powershell, execute o comando para verificar qual a versão do wsl habilitado para a distribuição que acabou de instalar:

```
wsl -l -v
```

![Versão do wsl]()

Caso a versão indicada seja a versão 1 utilize o seguinte comando para modificar a versão do wsl

```
wsl --set-version <Nome da distro> 2
```

Obs: Para realizar a mudança da versão do wsl é necessário executar o comando acima com a distro parada, assim, antes de realizar o passo anterior pare a execução do linux. 

&nbsp;

## Erros para realizar o download da distribuição linux

Pode ocorrer alguns problemas na realização do download da distribuição em ambientes com restrinções de segurança, nesse caso, podemos utilizar uma distribuição linux importada de uma arquivo **.tar** ou tentar realizar o download da distribuição na loja do windows pelo navegador de internet.

Vamos mostrar como fazer a instalação através da importação o arquivo .tar.

Com o powershell aberto em seu emulador de terminal, execute o comando para criar um diretório no seu local preferido para realizar a instalação da distribuição linux.

```
mkdir E:\wslDistroStorage\Ubuntu
```

Em seguida, exeute os comandos para realizar a importação da distro e sua execução.

```
wsl --import <DistroName> <InstallLocation> <InstallTarFile>
wsl -d <Nome da distribuição>
```

&nbsp;

## Instalação do docker

Agora vamos realizar a instalação do docker engine no ubuntu que acabamos de instalar no wsl.

Vamos adicionar o repositório do docker a lista de repositórios da distro. Primeiramente vamos atualizar a lista de pacotes e realizar a instalação de alguns pacotes necessários para realizar requisições http.

~~~Bash
 sudo apt-get update
 
 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
~~~

Agora vamos adicionar a chave GPG oficial do docker

~~~Bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
~~~

Vamos configurar o repositório com o comando a seguir:

~~~Bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
~~~

Agora vamos realizar a atualização da lista de repositórios e instalar o docker

~~~Bash
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
~~~

Terminada a instalação precisamos criar o grupo docker e adicionar o nosso usuário a ele para que não seja necessário utilizar a utilização de privilégios de administrador para executar o docker, assim,

~~~Bash
sudo groupadd docker
sudo usermod -aG docker $USER
~~~

Inicie o serviço docker no sistema

~~~Bash
sudo service docker start
~~~

Se tudo deu certo a imagem de teste do docker será executada sem erros

~~~Bash
sudo docker run hello-world
~~~

Mas claro que nem tudo são flores e podemos ter problemas no momento de executar o docker devido a problemas de conexão do daemon.

&nbsp;

## Erro ao tentar conectar o daemon do docker

![Error docker daemon]()

Das inumeras alternaivas para resolver o problemas disponíveis em fóruns na internet apenas uma realmente resolveu o problema. No caso especifico do Ubuntu 22.04 o pacote de iptables foi atualizado para uma versão mais recente e com isso essa atualização comprometeu endereçamento de ips para a conexção do daemon do docker.

Para resolver o problema é necessário alterar o pacote do iptables para a versão legacy, desse modo, podemos fazer isso com o comando abaixo:

``` 
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy 
```

em seguida, podemos executar o comando

```
dockerd
```

se tudo correr bem, nenhum erro sera indicado no terminal, assim, podemos terminar a execução do docker e executar o comando para iniciar o serviço e em seguida, executar o hello world.
