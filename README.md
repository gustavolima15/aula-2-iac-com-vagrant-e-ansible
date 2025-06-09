# Aula 2 - IAC com Vagrant e Ansible

Neste repositório, vamos aprender a usar o Vagrant e o Ansible para criar e gerenciar máquinas virtuais.
Também vamos aprender a usar o Ansible para fazer o deploy do site "mundo invertido"

## Tabela de conteúdos

- [Pré-requisitos](#pré-requisitos)
- [Passo a passo](#passo-a-passo)
- [Erros conhecidos](#erros-conhecidos)

## Pré-requisitos

- Instalação do VirtualBox
    - https://www.virtualbox.org/wiki/Downloads
- Instalação do Vagrant
    - https://developer.hashicorp.com/vagrant/downloads?product_intent=vagrant
- Instalação do Visual Studio Code
    - https://code.visualstudio.com


## Passo a passo 

1. Crie uma pasta/diretório para o projeto (Ex.: `aula-2-iac-com-vagrant-e-ansible`) em sua pasta/diretório de trabalho ou na pasta/diretório de seu usuário

    Se preferir, fazer pelo terminal:
    ```bash
    cd seu_diretorio_de_trabalho
    mkdir aula-2-iac-com-vagrant-e-ansible
    cd aula-2-iac-com-vagrant-e-ansible
    ```

2. Se você estiver no Windows Explorer, clique com o botão direito do mouse sobre a pasta/diretório criada e selecione "Open in Terminal"
3. Já dentro do terminal, execute o seguinte comando:
    ```bash
    vagrant init
    ```
    O comando irá gerar um arquivo chamado `Vagrantfile`
4. Execute o seguinte comando para editar o arquivo `Vagrantfile`:
    ```bash
    code .
    ```
5. Inclua as seguintes informações no arquivo `Vagrantfile`:
    ```ruby
    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    Vagrant.configure("2") do |config|
        config.vm.box = "ubuntu/focal64"
    
        config.vm.network "forwarded_port", guest: 80, host: 8080

        config.vm.provider "virtualbox" do |vb|
          vb.memory = "1024"
          vb.cpus = 1
          vb.name = "nginx - webserver"
        end

        config.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "playbook.yml"
        end
    end
    ```
    > [!NOTE]
    > Esse arquivo Vagrantfile é o arquivo declarativo que informa ao Vagrant como deve ser a máquina virtual que será criada e como deve ser configurada, usando o Ansible para fazer a configuração

6. Agora vamos criar o arquivo de playbook do Ansible:
    
    ```bash
    code playbook.yml
    ```
    > [!NOTE]
    > Esse arquivo playbook.yml é o arquivo declarativo que informa ao Ansible como deve ser o estado desejado do sistema, ou seja, o que o Ansible deve fazer para configurar a máquina virtual e garantir que ela esteja sempre neste estado

7. Agora inclua as seguintes informações no arquivo `playbook.yml`:
    
```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: Atualiza o cache do apt
      apt:
        update_cache: yes
      tags:
        - packages

    - name: Instala o Nginx
      apt:
        name: nginx
        state: present
      tags:
        - packages

    - name: Copia a página web para o diretório do Nginx
      copy:
        src: files/
        dest: /var/www/html
        owner: www-data
        group: www-data
        mode: '0644'
      notify:
        - Reiniciar Nginx

  handlers:
    - name: Reiniciar Nginx
      service:
        name: nginx
        state: restarted
```

8. Agora podemos iniciar o provisionamento da máquina virtual:
    ```bash
    vagrant up
    ```

9. Assim que a máquina virtual for provisionada, podemos acessar o site pelo navegador:
    
    `http://localhost:8080`


## Erros conhecidos

No Windows, caso você receba este erro do VirtualBox:

> [!CAUTION]
VT-x is not available. (VERR_VMX_NO_VMX)

Significa que o **Hyper-V** está habilitado e configurado como virtualizador padrão no Windows, pois ele e o VirtualBox não podem coexistir.

Para resolver isso, você precisa desabilitar o **Hyper-V**, para isso siga estes passos no Terminal:

```bash
bcdedit /set hypervisorlaunchtype off
```

Depois de desabilitar o **Hyper-V**, reinicie o computador e tente novamente.



