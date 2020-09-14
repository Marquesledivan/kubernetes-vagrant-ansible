- [Kubernetes Vagrant](#kubernetes-vagrant)
- [0. Instalar (ou atualizar) dependencias](#0-instalar-ou-atualizar-dependencias)
  * [OSX com homebrew (https://brew.sh/)](#osx-com-homebrew-httpsbrewsh)
  * [Windows com chocolatey](#windows-com-chocolatey)
  * [Linux, FreeBSD, OpenBSD, outros](#linux-freebsd-openbsd-outros)
  * [Vagrant plugins](#vagrant-plugins)
  * [A base box do vagrant](#a-base-box-do-vagrant)
- [1. Baixar esse repositório se ainda não o fez](#1-baixar-esse-repositório-se-ainda-não-o-fez)
- [2. Montando o cluster](#2-montando-o-cluster)
  * [Remover arquivo antigo de configuração](#remover-arquivo-antigo-de-configuração)
  * [Levantando as máquinas](#levantando-as-máquinas)
  * [Fazer reset se tiver sido criado anteriormente](#fazer-reset-se-tiver-sido-criado-anteriormente)
  * [Inicializar o nó master](#inicializar-o-nó-master)
  * [Configurar rede](#configurar-rede)
  * [Adicionar nós worker](#adicionar-nós-worker)
- [3. Configurar o kubconfig](#3-configurar-o-kubconfig)
  * [Pegar o novo arquivo de configuração](#pegar-o-novo-arquivo-de-configuração)
  * [Apontar o `kubectl` para usá-lo](#apontar-o-kubectl-para-usá-lo)
- [4. Desligando ou reiniciando](#4-desligando-ou-reiniciando)


# Kubernetes Vagrant

Esse projeto entrega 3 máquinas virtuais Virtualbox rodando Ubuntu 20.04 com `kubeadm` e dependências instaladas mais as instruções para montar seu cluster `Kubernetes` multinode (1 master + 2 workers).

Utilizando o ansible como provision, qualquer alteração do no workload deve se analizar o playbook.

Utilizando para o join no cluster o nginx com o comando: ## kubeadm token create --print-join-command >> /var/www/html/index.nginx-debian.html

Foi atualizado e testado com Kubernetes 1.19, Weave latest (2.7.0) e Docker 19.03.

# 0. Instalar (ou atualizar) dependencias

Instalar Virtualbox, Vagrant, Ansible os plugins vagrant-cachier e vagrant-vbguests e a kubernetes cli.

## OSX com homebrew (https://brew.sh/)

~~~bash
brew cask upgrade virtualbox vagrant
brew upgrade kubernetes-cli
~~~

ou

~~~bash
brew cask install virtualbox vagrant
brew install kubernetes-cli
~~~
## Linux, FreeBSD, OpenBSD, outros

Você deve saber o que está fazendo. Mas aqui seguem alguns links para ajudar:

- https://www.virtualbox.org/wiki/Downloads
- https://www.vagrantup.com/downloads.html
- https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
- https://kubernetes.io/docs/tasks/tools/install-kubectl/

## Vagrant plugins

Primeiro removemos caso tenha instalado os plugins anteriormente e instale um por vez para evitar conflito de dependencia de versão.

~~~bash
vagrant plugin uninstall vagrant-share
vagrant plugin uninstall vagrant-cachier
vagrant plugin uninstall vagrant-hostmanager
vagrant plugin uninstall vagrant-vbguest
vagrant plugin install vagrant-cachier
vagrant plugin install vagrant-vbguest
~~~

# 1. Baixar esse repositório se ainda não o fez

Primeiro clonamos o projeto:

~~~bash
git clone https://github.com/Marquesledivan/kubernetes-vagrant-ansible
cd kubernetes-vagrant-ansible
~~~

# 2. Montando o cluster

## Remover arquivo antigo de configuração

Remover arquivo de configuração antigo criado previamente se existir

~~~bash
rm -f ./kubernetes-vagrant-config
~~~

## Levantando as máquinas

Inicie as máquinas ou reprovisione, leva cerca de 19 minutos em uma conexão de internet comum.

~~~bash
vagrant up k8smaster --provision; vagrant up k8snode1 --provision; vagrant up k8snode2 --provision
~~~

Mas não faça em paralelo isso pode causar problemas devido ao cache do apt não estar pronto ainda. Para provisionar as máquinas em paralelo você pode usar uma sessão shell para cada máquina e desligar o plugin `vagrant-cachier`. Apenas comente a linha `if Vagrant.has_plugin?("vagrant-cachier")` e a linha `end`  correspondente no `Vagrantfile`


Se quiser rodar pods no nó master (não recomendado)

~~~bash
vagrant ssh k8smaster -c "kubectl taint nodes --all node-role.kubernetes.io/master- "
~~~

Colocar rótulo de worker (opcional)

~~~bash
vagrant ssh k8smaster -c "kubectl label node k8snode1 node-role.kubernetes.io/node="
~~~

~~~bash
vagrant ssh k8smaster -c "kubectl label node k8snode2 node-role.kubernetes.io/node="
~~~

# 3. Configurar o kubconfig

## Pegar o novo arquivo de configuração

Copiar arquivo de configuração do nó master para a pasta compartilhada

~~~bash
vagrant ssh k8smaster -c "cp ~/.kube/config /vagrant/kubernetes-vagrant-config"

export KUBECONFIG=$PWD/kubernetes-vagrant-config

kubectl get pods -A

~~~

# 4. Desligando ou reiniciando

Para desligar rodamos `vagrant halt`.

Quando precisar do cluster novamente rode o [passo 2](#2-montando-o-cluster) e o [passo 3](#3-configurar-o-kubconfig) de novo, vai rolar o reprovisionamento mas muito mais rápido que da primeira vez.
