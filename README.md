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

Alteramos o nosso playbook para que agora ele execute o comando `apt` como root (através do comando `become`):

```yml
---
- hosts: all
  tasks:
  - apt: 
      name: php5
      state: latest
    become: yes
```

Quando executarmos novamente ele irá mostrar que está executando o comando e seu devido status:

```bash
lucaspolo@nb:/storage/cursos/alura/ansible$ ansible-playbook provisioning.yml -u vagrant -i hosts --private-key .vagrant/machines/wordpress/virtualbox/private_key

PLAY [all] ************************************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************************
ok: [172.17.177.40]

TASK [apt] ************************************************************************************************************************************************************************************
changed: [172.17.177.40]

PLAY RECAP ************************************************************************************************************************************************************************************
172.17.177.40              : ok=2    changed=1    unreachable=0    failed=0
```

Agora que conseguimos instalar o PHP, podemos prosseguir e instalar o Apache 2 e o modphp apenas incluindo eles no provisionamento. Aproveitaremos e incluiremos também nomes descritivos:

```yml
---
- hosts: all
  tasks:
  - name: 'Instala o PHP 5 na última versão'
    apt: 
      name: php5
      state: latest
    become: yes
  - name: 'Instala o Apache 2'
    apt:
      name: apache2
      state: latest
    become: yes
  - name: 'Instala o modphp'
    apt:
      name: libapache2-mod-php5
      state: latest
    become: yes
```

Com isto, basta rodar novamente o ansible-playbook e ele irá instalar as dependências. Confira em (http://172.17.177.40) se a página default do Apache aparece.