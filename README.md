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

## Aula 3: Boas práticas

Como podemos ver, temos código repetido dentro do nosso arquivo. Para ajudar nisso é possível iterar sobre uma lista passada para dentro do playbook para que possamos repetir o mesmo comando para diversos itens:

```yml
---
- hosts: all
  tasks:
  - name: 'Instala pacotes de dependência do sistema operacional'
    apt:
      name: "{{ item }}"
      state: latest
    become: yes
    with_items:
      - php5
      - apache2
      - libapache2-mod-php5
      - php5-gd
      - libssh2-php
      - php5-mcrypt
      - mysql-server-5.6
      - python-mysqldb
      - php5-mysql
```

Assim ele instalará todos os pacotes necessários para executarmos nosso trabalho.

Agora, podemos também reduzir a necessidade de passar o usuário e caminho da chave privada quando executamos `ansible-playbook`, para isto basta alterarmos o arquivo hosts:

```yml
[wordpress]
172.17.177.40 ansible_user=vagrant ansible_ssh_private_key="/storage/cursos/alura/ansible/.vagrant/machines/wordpress/virtualbox/private_key"
```

Desta forma o comando ficará bem mais enxuto:

```bash
ansible-playbook provisioning.yml -i hosts
```

## Aula 4: Preparando o banco de dados

Para criar o banco de dados é bem simples, basta incluirmos um novo comando dentro do playbook, passando o nome do banco, o usuário e o state:

```yml
---
- hosts: all
  tasks:
  ...outras tasks
  - name: 'Cria o banco do MySQL'
    mysql_db: 
      name: wordpress_db
      login_user: root
      state: present
```

Após rodarmos novamente o ansible-playbook podemos conferir a criação do banco de dados.

Para criarmos o usuário do banco de dados também criaremos uma nova task, desta vez usando o módulo `mysql_user`:

```yml
- hosts: all
  tasks:
  ...outras tasks
  - name: 'Cria o usuário do banco de dados'
    mysql_user:
      login_user: root
      name: wordpress_user
      password: 12345
      priv: 'wordpress_db.*:ALL'
      state: present
```