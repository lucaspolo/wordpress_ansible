# Curso de Ansible

Acompanhando curso de [Ansible da Alura](https://www.alura.com.br/curso-online-infraestrutura-como-codigo-com-ansible)

## Aula 1

Executar o primeiro comando Ansible:

```bash
ansible -vvvv wordpress -u vagrant --private-key .vagrant/machines/wordpress/virtualbox/private_key -i hosts -m shell -a 'echo Hello, world'
```

Desta forma, baseado na máquina Vagrant criada e no arquivo host, é passado um comando para a máquina dando um "Hello, world"

```bash
172.17.177.40 | SUCCESS | rc=0 >>
Hello, world

lucaspolo@nb:~/storage/cursos/alura/ansible$
```

## Aula 2: Criação do primeiro playbook

Definiremos o nosso playbook através do arquivo que criaremos `provisioning.yml`.

```yml
---
- hosts: all
  tasks:
    - shell: 'echo hello > /vagrant/world.txt'
```

Assim, agora o comando que iremos executar para que ele leia este arquivo e trabalhe é o seguinte:

```bash
lucaspolo@nb:/storage/cursos/alura/ansible$ ansible-playbook provisioning.yml -u vagrant -i hosts --private-key .vagrant/machines/wordpress/virtualbox/private_key

PLAY [all] ************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************
ok: [172.17.177.40]

TASK [shell] **********************************************************************************************************************************************************************************
changed: [172.17.177.40]

PLAY RECAP ************************************************************************************************************************************************************************************
172.17.177.40              : ok=2    changed=1    unreachable=0    failed=0
```

Assim, agora acessando a máquina com o comando `vagrant ssh` e verificando o diretório /vagrant podemos ver que foi criado o arquivo world.txt. Nossa receita funcionou e podemos seguir adiante com a configuração.