# Openshift TestDrive para Infra-Estrutura
Openshift TD para Infra 

## Exercicio 1 -  Conhecendo o ambiente ( 30 minutos )
Acessar a interface Web e acessar a sua instancia. Foi criado uma instancia para instalacao de um Openshift All-in-One. 

Dar comandos para reconhecer o ambiente. 

Ex:

    ls -la /etc/ansible/hosts

Ou ver se o docker esta rodando

    docker info

Ou qualuer outros comandos que julgar necessários para conhecer o ambiente

## Exercicio 2 - Inventario Openshift ( 30 minutos )

O inventario do Openshift é a peca essencial de uma instalacao. Juntamente com os pre-requisitos. Vamos revisar o inventario em:

    vi /etc/ansible/hosts

## Exercicio 3 - Os pre-requisitos ( 30 minutos )
Toda instalacao de Openshift é feita com sucesso se atendar para os prerequisitos. Vamos checar a documentacao e avaliar um conjunto de playbooks que pode ajudar. Voce pode fazer o seu!

[https://docs.okd.io/3.11/install/index.html]

Desmistificando a instalacao, um conjunto de playbooks de ajuda:

[https://github.com/redhatbsb/ocp-playbooks]

## Exercicio 4 - Instalando o Openshift ( 45 minutos a 1 hora )
Em sua maquina, verifique se esta como usuario centos
    
    id

O resultado esperado é algo como:

    [centos@ip-172-31-20-224 openshift-ansible]$ id
    uid=1000(centos) gid=1000(centos) groups=1000(centos),4(adm),10(wheel),190(systemd-journal) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

Verifique se no home do usuario existe o diretorio "openshift-ansible"

    cd ~/openshift-ansible

Esse é o instalador oficial do Openshift 3.x

[https://github.com/openshift/openshift-ansible]

Para este TD, ja rodamos o pre-requisitos. Mas voce pode rodar novamente.

    ansible-playbook playbooks/prerequisites.yml

Para Instalacao do Cluster, basta rodar o deploy_cluster

    ansible-playbook playbooks/deploy_cluster.yml


## Exercicio 5 - Acessando sua console (15 minutos)

Voce pode agora acessar a sua console do Openshift. Na sua interface Web do Test-Drive va em Ferramentas -> Acesso a sua instancia. La procure por "IP Publico Instancia:"

Entao digite no browser:
    https://ippublicodasuainstancia:8443/

## Exercicio 6 - Dando uma bombada no superadmin

Voce viu que está sem possibilidade de ver algumas coisas... vamos bombar seu usuário...

    oc adm policy add-cluster-role-to-user cluster-admin superadmin

## Exercicio 7 - Decidindo nosso caminho (30 a 45 minutos)

Faca o deployment de um app de votacao que iremos utilizar para decidir se iremos fazer o path Openshift padrao ou se seguimos dando mais enfase em infra-estrutura. Faca a enquete com duas opcoes:

1. Path Normal 
2. Path Infra ++

Ambos os "paths" sao semelhantes. O "Path Normal" possui funcoes de Dev/Infra e o "Path Infra++" possui algumas funcoes interessantes como criação de PVs e Dynamic Provisioning.

Depois, publique no etherpad para podermos trocar ideias com os colegas e escolher o melhor pool.

## Exercicio 8 - Seguir com o path

Aqui seguiremos os exercicios padroes e pulando alguns por conta do tempo.

Entre os exercicios, podemos precisar realizar alguns fixes, entre eles:

1. Metricas

Edite o arquivo de inventario (/etc/ansible/hosts) e adicione a seguinte linha:

    openshift_metrics_server_install=true


E rodar o comando para instalar metricas:

    cd ~/openshift-ansible/ && ansible-playbook playbooks/metrics-server/config.yml


2. Volumes Persistentes precisarao de NFS

## Exercicio 9 - Em caso de Infra++ - Volumes Persistentes

Veja os PVs criados:

    oc get pv

Este comando é cluster wide, ou seja, mostra todos os volumes persistentes disponiveis, alocados ou liberados.

Crie um PVC via console ou via linha de comando e liste-o dentro do seu projeto

    oc get pvc

## Exercicio 10.1 - Em caso de Infra++ - Volumes Persistentes Dinamicos - Solucao Automatica


1. exportar as sguintes variaveis de ambiente

```
export AWS_ACCESS_KEY_ID=<key_ID>
export AWS_SECRET_ACCESS_KEY=<secret_key>
```

2. Editar o arquivo /etc/ansible/hosts com o seguinte conteudo adicional
```
openshift_cloudprovider_aws_access_key="{{ lookup('env','AWS_ACCESS_KEY_ID') }}"
openshift_cloudprovider_aws_secret_key="{{ lookup('env','AWS_SECRET_ACCESS_KEY') }}"
openshift_clusterid=openshift
openshift_cloudprovider_kind=aws
```

3. Mandar um uninstall sem pena

```
cd ~/openshift-ansible
ansible-playbook playbooks/adhoc/uninstall.yml
```

4. Mandar um novo install 
```
ansible-playbook playbooks/deploy_cluster.yml
```

## Exercicio 10.2 - Em caso de Infra++ - Volumes Persistentes Dinamicos - Solucao Manual (Nao testada 100%)

Vamos configurar Dynamic Provisioning usando AWS

1. Criar o Arquivo /etc/origin/cloudprovider/aws.conf com o seguinte conteudo

```
[Global]
Zone = sa-east-1c
```

2. Editar o arquivo /etc/origin/master/master-config.yaml e colocar o seguinte conteudo: 

Porem, sugere-se fazer a copia do arquivo original

    cp /etc/origin/master/master-config.yaml /etc/origin/master/master-config.yaml-original
    vi /etc/origin/master/master-config.yaml

```
kubernetesMasterConfig:
  ...
  apiServerArguments:
    cloud-provider:
      - "aws"
    cloud-config:
      - "/etc/origin/cloudprovider/aws.conf"
  controllerArguments:
    cloud-provider:
      - "aws"
    cloud-config:
      - "/etc/origin/cloudprovider/aws.conf"
```



1. Depois também fazer a configuracao do /etc/origin/node/node-config.yaml

Porem, também sugere-se fazer a copia do arquivo original

    cp /etc/origin/node/node-config.yaml /etc/origin/node/node-config.yaml-original
    vi /etc/origin/node/node-config.yaml

```
kubeletArguments:
  cloud-provider:
    - "aws"
  cloud-config:
    - "/etc/origin/cloudprovider/aws.conf"
```

4. Configurar os arquivos  /etc/origin/master/master.env e /etc/sysconfig/origin-node com o seguinte conteudo:

```
AWS_ACCESS_KEY_ID=<key_ID>
AWS_SECRET_ACCESS_KEY=<secret_key>
```

Com os valores a serem fornecidos pelo instrutor.

5. Restartar a master e nodes com os seguintes comandos:

```
master-restart api
master-restart controllers
systemctl restart atomic-openshift-node
```

6. Criar a Storage Class para Dynamic provisioning

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: awsebs
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  zone: sa-east-1c
  encrypted: "false"  
  fsType: ext4
```




