Role Name
=========

**PowerVC_deploy_lpar** est un playbook Ansible idempotent permettant la creation et mise a jour de la configuration de LPAR via PowerVC. 

Prerequis
---------

Ce playbook utilse les API PowerVC (REST API openstack) pour provisionner des LPAR.  
L'acces a un serveur powerVC avec les credential est donc indispensable pour utiliser ce playbook.

Definition des roles
--------------------

Ce playbook s'appuie sur un set de role permettant de ne pas dupliquer inutilment du code.

nom du role                 | definition du role
---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
__PowerVC_deploy_lpar__    | Role principal idempotent permettant de créer/deployer une LPAR ou de mettre a jour sa configuration si celle ci existe deja.
__PowerVC_get_token__      | Permet de recupérer un token powerVC/openstack a partir d'un login/password.
__PowerVC_credential__     | Dans le cas ou l'on ne souhaite pas stocker les variables `PowerVC_USER` ou `PowerVC_PWD` dans des fichiers de configuration.
__PowerVC_get_ids__        | Recupere des UID openstack a partir de nom de ressources (controlé par la variable `ID_type`) ex: tenant_id à partir d'un nom de projet (varibale `PowerVC_Project`) etc ...
__PowerVC_create_volume__  | Role permettant la creation d'un volume additionel
__PowerVC_attach_volume__  | Attache un volume additionel à un Serveur (LPAR)
__PowerVC_wait4_healthOK__ | Boucle de synchronisation permettant d'attendre qu'une LPAR passe a l'etat Health OK (RMC UP_AND_RNNING)
__PowerVC_wait4_ssh__      | Boucle de synchronisation permettant d'attendre qu'une LPAR soit accessible par SSH (LPAR UP_AND_RUNNING)

Example d'utilisation du Playbook
---------------------------------

Vous devez au prealable avoir une image capturée avec PowerVC. Par exemple `RHEL76`


```
# ansible-playbook PowerVC_deploy_lpar.yml [-l hostname] [-e IMAGE=RHEL76]
```

------------------------------------------------------------------------------------------------

PowerVC_deploy_lpar
-------------------

Ce role verifie d'abord si la LPAR a créer existe deja. 
Dans la negative, la LPAR sera créée via la task `create_lpar.yml`
Dans la positive, la configuration de le LPAR sera mise a jour avec les nouvelles valeures (si besoin) via la task `update_lpar.yml`

### Variables du role


Les Variables avec (*) sont obligatoires 

Nom de la variable          | Description
----------------------------|----------------------------------------------------------------------------------------------------------------------------------
__PowerVC_URL*__            | PowerVC URL ex: `https://10.3.54.72`
__PowerVC_USER__            | PowerVC User login (sera demandé en prompt interactif si non fournit)
__PowerVC_PWD__             | PowerVC User password (sera demandé en prompt interactif si non fournit)
__PowerVC_Project*__        | Projet PowerVC dans lequel créer la LPAR (openstack tenant)
__network*__                | Variable definissant les nom des reseaux et IPs a definir sur la LPAR (type dict.)
__STOR_CONN_GROUP*__        | PowerVC Storage Connectivity Group
__STOR_TEMPLATE*__          | Storage Template correspondant au Storage Connectivity Group
__AGGREGATES*__             | PowerVC Host Group sur lequel créer la LPAR
__dedicated_proc__          | Utilisation de processeur dediés (default: false)
__vcpus*__                  | Nombre de VCPU pour la LPAR
__min_vcpus__               | Nombre de VCPU minimum pour la LPAR (DLPAR) (defaut: 1)
__max_vcpus__               | Nombre de VCPU maximum pour la LPAR (DLPAR) (defaut: vcpus * 2)
__proc_units*__             | Nombre de ProcUnits(EC) pour la LPAR
__min_proc_units__          | Nombre de minimum de ProcUnits(EC) pour la LPAR (defaut: 0.1)
__max_proc_units__          | Nombre de maximum de ProcUnits(EC) pour la LPAR (defaut: proc_units * 2)
__ram_mb*__                 | Nombre de MB de Memoire RAM pour la LPAR (default: 4096)
__min_mem__                 | Nombre minimum de MB de Memoire RAM pour la LPAR (DLPAR) (defaut: 2048)
__max_mem__                 | Nombre minimum de MB de Memoire RAM pour la LPAR (DLPAR) (defaut: ram_mb * 2)
__disk_gb__                 | Taille du disque OS(boot) en GB
__IMAGE__                   | PowerVC image a deployer.
__PAUSE_IN_SEC__            | Pause avant de lancer la boucle verifiant le statut du provisioning de la LPAR (defaut: 30 secondes)
__nb_retry__                | Nombre d'essaie de verification du statut du provisionning avant de la declarer en ECHEC (defaut: 100)
__nb_delay__                | Delai entre chaque verification de statut du provisioning (defaut: 20 secondes)
__additional_volume_group__ | Volumes additionels a créer au niveau d'un group_var de machine ansible
__additional_volume_host__  | Volumes additionels a créer au niveau d'un host_var de machine ansible
__rsa_pub_key__             | Clé ssh publique a deployer sur la LPAR pour une authentification sans password. (defaut: contenu du fichier `~/.ssh/id_rsa.pub`)

*exemple de fichier de variable pour deploiement:*    

```
PowerVC_URL: https://10.3.54.72
PowerVC_USER: root
PowerVC_PWD: powerlinux
PowerVC_Project: ibm-default

#PAUSE_IN_SEC: 30
#nb_retry: 100
#nb_delay: 20
#nb_timeout: 900

vcpus: 2
proc_units: 0.4
ram_mb: 2048
disk_gb: 50
IMAGE: RHEL76LE
STOR_CONN_GROUP: "Any host, all VIOS"
STOR_TEMPLATE: SVCbase
AGGREGATES: "Default Group"

network:
  - name: VLAN354
    ip: 10.3.54.201
  - name: VLAN719
    ip: 10.7.19.201

additional_volume_host:
  - name: "{{ inventory_hostname }}_data1"
    size: 10
    StorageTemplate: "{{ STOR_TEMPLATE }}"
  - name: "{{ inventory_hostname }}_data2"
    size: 10
    StorageTemplate: "{{ STOR_TEMPLATE }}"
```

> Note: **Modification cloud-init**   
Il est possible de modifier le cloud-init lors du deploiement. Cette configuration est stocké au niveau d'un template du role. `templates/cloud-config.j2`. Il est possible de faire evoluer et variabiliser ce fichier pour l'adapter aux futures besoins.

```
#cloud-config
manage_etc_hosts: true
ssh_authorized_keys:
 - {{ rsa_key_public }}
runcmd:
 - firewall-cmd --zone=public --add-port=657/tcp --permanent
 - firewall-cmd --zone=public --add-port=657/tcp --permanent
 - firewall-cmd --reload
final_message: "Deploiement is Done

```


---

PowerVC_get_token
-------------------

Ce role recupere un token apres authentification aupres de PowerVC.
Ce role utile les variables `PowerVC_USER` et `PowerVC_PWD` pour s'authentifier. Si celles si ne sont pas renseignées, elles seront demandées en prompt interactif a l'utilisateur via le role `PowerVC_credential` 

### Variables du roles

Les Variables avec (*) sont obligatoires  

Nom de la variable   | Description
---------------------|---------------------------------------------------
__PowerVC_URL*__     | URL de connexion PowerVC (https://<IP_PowerVC>)
__PowerVC_USER__     | PowerVC Username
__PowerVC_PWD__      | PowerVC Password
__PowerVC_Porject*__ | Projet powerVC (tenant) sur lequel s'authentifier.

---

PowerVC_credential
-------------------

Ce role permet de créer les variables `PowerVC_USER` et `PowerVC_PWD` pour une authentification interactive. (à utiliser avant le role `
PowerVC_get_token`)

---

PowerVC_get_ids
---------------

Ce role permet de recupérer les ID de certain objet openstack a partir de leur nom. 
Sont pris en charge: `tenant`, `server`, `host_aggregate`, `image`, `network`, `storage_cg`, `volume`, `volumeAttachment`

Le type d'ID a recupérer doit etre specifié dans une variable nommé `ID_type`
L'ID retourné sera stocké dans une variable nommée `<ID_type>_id`
exemple: ID_type=tenant -> ID recupéré dans `tenant_id`

La tache `main.yml` se charge de lancer requete vers l'objet demandé via des sous tache `get_<ID_type>_id.yml` 

*exemple d'utilisation:*  

```
# Recuperation du Project ID de facon basique
  - name: Get Project/tenant ID
    include_role:
      name: PowerVC_get_ids
    vars:
      ID_type: tenant

# Recuperation des tenant_id et server_id via une loop
  - name: Get PowerVC Ids
    include_role: 
      name: PowerVC_get_ids
    vars:
      ID_type: "{{ item }}"
    loop:
      - tenant
      - server
```

---

PowerVC_create_volume
---------------------

Ce role créée les volumes additionnel pour la LPAR.  
Les characteristiques des volumes a créer peuvent se trouver dans 2 variables `additional_volume_group` definie dans un group_var et `additional_volume_host` definie dans un host_var.  
La variable `additional_volume` utilisé dans la suite du playbook est le resultat de la concatenation des ces 2 variables pour chaque LPAR. (ce qui permet de créer un set de volume pour le group, et d'y ajouter un set de volumes specifique par host)  


```
# Group_var contenant un la definition de 2 volumes additionels.
additional_volume_group:
  - name: "{{ inventory_hostname }}_data1"
    size: 10
    StorageTemplate: "SVC base template"
  - name: "{{ inventory_hostname }}_data2"
    size: 5
    StorageTemplate: "SVC base template"
```

---

PowerVC_attach_volume
---------------------

Ce role attache les volumes additionnel créés precedement à la LPAR.  
Ce role ne peut s'executer que si la connexion RMC est etablie entre la partition (LPAR) et la HMC. En PowerVC, cela se traduit par un "health status=OK" qui peut etre verifier a l'aide du role `PowerVC_wait4_healthOK`.  

---

PowerVC_wait4_healthOK
----------------------

Ce role est une boucle d'attente permettant de s'assurer que le connexion RMC entre la LPAR et la HMC est etablie (PowerVC health status OK). Cette connexion est indispensable pour permettre des actions de type modification dynamique de configuration des LPAR (add CPU, proc_units, volumes ...)

### Variables du roles  

Nom de la variable | Description
-------------------|-----------------------------------------------------------------------------------
__nb_retry__       | nombre maximum de boucles (essaies) avant de passer en etat FAILED. (default: 100)
__nb_delay__       | delai, en seconde, entre chaque boucle (default: 20)

---
 
PowerVC_wait4_ssh
----------------------

Ce role est une boucle d'attente permettant de s'assurer que la LPAR est demarrée et accessible par SSH.  

### Variables du roles  

Nom de la variable | Description
-------------------|-------------------------------------------------------------------------
__nb_delay__       | delai, en seconde, avant la premiere tentative de connexion (default: 1)
__nb_timeout__     | temps d'attente maximum en seconde (default: 60)