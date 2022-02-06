## Configurando serviços Apache + PHP na AWS

---

### Documentação AWS EC2
<https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/concepts.html>


---

### Tutorial configurar um servidor LAMP
<https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/install-LAMP.html>


---


### Configurações iniciais na instância Amazon AMI

depois de logar na instância com o ec2-user

- Atualizar os pacotes de softwares
    > `sudo yum update -y`
- Habilitar a instalação do PHP 7.4
    > `sudo amazon-linux-extras enable php7.4`
- Limpar o cache armazenado no sistema
    > `sudo yum clean metadata`
- Instalar o Apache e o PHP
    > `sudo yum install httpd mod_ssl php7.4 php7.4-mysqlnd`

---


### Adicionando um Virtual Host
#### Vou simular que estou criando um Virtual host para o site **www.exemplo.com.br**

- Na pasta `/var/www` criar uma pasta que vai conter a instalação do site:
    >   `sudo mkdir /var/www/exemplo`
- Na pasta `/etc/httpd/conf.d`, é onde fica as configurações de Virtual Host do Apache, criar um novo arquivo com a extenção `.conf`:
    > `sudo nano exemplo.conf`
- Ao abrir a interface do `nano` digite a seguinte configuração, lembrando de substituir o endereço da pasta e o domínio do site que deseja criar:

            <Directory "/var/www/exemplo">
                    Options Indexes FollowSymLinks
                    AllowOverride All
            </Directory>

            <VirtualHost *:80>
                    DocumentRoot /var/www/exemplo
                    ServerName www.exemplo.com.br
                    ServerAlias exemplo.com.br
                    ErrorLog "logs/exemplo-error.log"
                    CustomLog "logs/exemplo-access.log" common
            </VirtualHost>

- Agora precisa reiniciar ou iniciar o apache e depois testar no navegador para ter certeza que está funcionando. Para reiniciar o apache:
    > `sudo service httpd restart`
- Configura o Apache para iniciar junto com a inicialização do sistema:
    > `sudo systemctl enable httpd`

---

### Adicionar o Swap na instância
#### no exemplo vou adicionar 4GB de Memória Swap
#### Documentação de referência para criação de Swap <https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-centos-7>

- Para saber se na instância já existe o Swap
    > `swapon -s`
- Para verificar Quanto de memória está sendo utilizada no Swap
    > `free -m`
- Verifique quanto de memória Física está disponível para que possa separar uma parte para o Swap:
    > `df -h`
- Criar um arquivo de Swap com 4GB, modifique a memória de acordo com a necessidade (Esse processamento pode demorar um pouco, às vezes até 10 minutos):
    > `sudo dd if=/dev/zero of=/swapfile count=4096 bs=1MiB`
- Configura a permissão do Swap file apenas para o usuário Root
    > `sudo chmod 600 /swapfile`
- Para verificar se a permissão está ok (-rw------- 1 root root 4.0G Oct 30 11:00 /swapfile):
    > `ls -lh /swapfile`
- Configurando como um arquivo de Swapspace no sistema:
    > `sudo systemctl enable httpd`
 - Habilitando para ser usado:
    > `sudo swapon /swapfile`
- Para verificar se tudo está certo:
    > `swapon -s`
- Para deixar o Swapfile permanente precisa configurar no arquivo /etc/fstab
    > `sudo nano /etc/fstab`
- Adicione a linha embaixo de outras configurações que possam existir

        /swapfile   swap    swap    sw  0   0

*Para otimizar o arquivo o uso do Swap, veja a documentação no link que está no título*

---

### Adicionando Certficado SSL para usar o protocolo HTTPS
#### Vamos utilizar o Certbot / LetsEncrypt para a certificação <https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/SSL-on-amazon-linux-2.html#letsencrypt>
- Execute os comandos na sequência para configurar o repositório de instalação
    > `sudo wget -r --no-parent -A 'epel-release-*.rpm' https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/`

    > `sudo rpm -Uvh dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-*.rpm`

    > `sudo yum-config-manager --enable epel*`

    > `sudo yum repolist all`

- Para instalar o Certbot
    > `sudo yum install -y certbot python2-certbot-apache`

    > `sudo certbot`

- Na Sequência Aceite os Termos. Na primeira vez que for usar ele deve pedir um e-mail para receber alguma notificação;
- Irá ser listado uma lista de domínio instalado para decidir qual domínio deseja a instalação do Certificado;
- Se ao finalizar pedir para Forçar o Redirecionamento, aceite.
- Agora configure a renovação automática à partir do Crontab. OBS, ele usa o **VI** para editar:
    > `sudo crontab -e`
- Verifique qual a melhor hora do dia para rodar o script de renovação e configura no Crontab, no exemplo a seguir está configurado para rodar todos os dias às 01hs e às 13hs: 

        0      1,13    *       *       *       root    certbot renew --no-self-upgrade






