# projet_ansible_grp3
Mise en situation 1

#### Informations de connexion
GRP3 :

   - [poecdevops.grp3.dawan.training](poecdevops.grp3.dawan.training)   
   - user : stagiaire
   - mdp : P0ec2021!
   - sudo avec mot de passe

---

## 1. Préparation des dépôts Git distants

- Création du dépôt sur GitHub
- Ajout de clé SSH sur GitHub
- Clone du dépôt: `git clone git@github.com:davidchdebray/projet_ansible_grp3.git`
- Connexion au serveur Debian *poecdevops.grp3.dawan.training*
   - `ssh-copy-id -i /home/stagiaire/.ssh/id_ed25519.pub stagiaire@poecdevops.grp3.dawan.training`


## 2. Démarrage environnement Ansible et projet Git Ansible

### Fichiers de configuration Ansible 

**Fichier: *Inventory***

```ini
[local]
manager ansible_connection=local

[nodes:children]
debian

[ubuntu]
manager

[debian]
hedgedoc ansible_host=poecdevops.grp3.dawan.training
 
[chafea]
chafeatest ansible_host=192.168.1.20

[david]
davidtest ansible_host=192.168.3.3

[vincent]
vincenttest ansible_host=192.168.1.56
```

**Fichier: *ansible.cfg***

[defaults]
inventory = ./inventory
remote_user = ansible


**Fichier: *bootstrap_playbook.yml***

```yml

---

- name: PLAY creation utilisateur ansible
  # Variabilisation de la valeur hosts (pattern) pour pouvoir la définir au lancement du play
  hosts: "{{ cible | default('all') }}"

  tasks:

    # Déclaration d'une task qui appelle le module user afin de disposer d'un user ansible
    - name: Utilisation du module user pour creer ansible
      ansible.builtin.user:
        name: ansible
        shell: /bin/bash
        # Utilisation d'une variable pour permettre une distinction du groupe désiré
        groups: "{{ grp_sudo | default('sudo') }}"
    # Déclaration d'une seconde task qui appelle le module authorized_key pour pousser une clé publique pour le user ansible
    - name: Module authorized_key pour deployer la clé publique chez ansible
      ansible.posix.authorized_key:
        user: ansible
        state: present
        key: "{{ lookup('file', '/home/stagiaire/.ssh/id_ed25519.pub') }}"
    # Déclaration d'une troisieme task pour générer un fichier de config sudo dédié au user ansible
    - name: Module copy pour s'assurer q'une fichier sudo pour ansible soit présent avec un contenu précis
      ansible.builtin.copy:
        dest: '/etc/sudoers.d/ansible'
        content: 'ansible ALL=(ALL:ALL) NOPASSWD: ALL'
        backup: yes
        owner: root 
        group: root
        mode: 0440
        validate: /usr/sbin/visudo -csf %s
...
```


### Execution du *bootstrap_playbook*

- `ansible-playbook bootstrap_playbook.yml -b --user stagiaire --ask-become-pass -e "cible=hedgedoc"`

On vérifie que les changements ont bien étés fait:

`ansible-playbook bootstrap_playbook.yml -b -e "cible=hedgedoc"` => OK




## 3. Déploiement serveur Docker

### Ajout du rôle Docker

````bash
mkdir roles && cd roles
git clone https://github.com/psable/ansible-role-docker
```

### 1. Créer un playbook deploy-docker.yml qui appelera le role ansible-role-docker__0.0.1

Contenu du playbook *deplay-docker.yml*:

```yml
---
- name: PLAY Apelle le rôle ansible-role-docker
  hosts: "{{ cible | default('nodes') }}"

  roles:
    - role: ansible-role-docker__0.0.1
      become: yes
```

### 2. Pour tester ce playbook, le faire d'abord sur une VM locale de dev puis une fois sur la machine distante projet Debian

Sur la VM de test:




### 3. Valider l'installation docker, vérifier la présence de docker-compose

### 4. Commit le code et pusher sur le dépôt git distant

### 5. Réaliser l'opération sur la VM projet Debian


