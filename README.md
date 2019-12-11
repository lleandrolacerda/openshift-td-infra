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

## Exercicio 3 - Instalando o Openshift ( 45 minutos a 1 hora )
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


## Exercicio 4 - Acessando sua console (15 minutos)

Voce pode agora acessar a sua console do Openshift. Na sua interface Web do Test-Drive va em Ferramentas -> Acesso a sua instancia. La procure por "IP Publico Instancia:"

Entao digite no browser:
    https://ippublicodasuainstancia:8443/


## Exercicio 5 - Decidamos nosso caminho (30 a 45 minutos)

Faca o deployment de um app de votacao que iremos utilizar para decidir se iremos fazer o path Openshift padrao ou se seguimos dando mais enfase em infra-estrutura. Faca a enquete com duas opcoes:

1. Path Normal 
2. Path Infra ++

Ambos os "paths" sao semelhantes. O "Path Normal" possui funcoes de Dev/Infra e o "Path Infra++" possui algumas funcoes interessantes como criação de PVs e Dynamic Provisioning.

Depois, publique no etherpad para podermos trocar ideias com os colegas e escolher o melhor pool.

## Exercicio 6 - Seguir com o path

Aqui seguiremos os exercicios padroes e pulando alguns por conta do tempo.

## Exercicio 7 - Em caso de Infra++ - Volumes Persistentes

Veja os PVs criados:

    oc get pv

Este comando é cluster wide, ou seja, mostra todos os volumes persistentes disponiveis, alocados ou liberados.

Crie um PVC via console ou via linha de comando e liste-o dentro do seu projeto

    oc get pvc

