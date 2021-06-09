---
title: Comment utiliser Terraform sur le Public Cloud OVHcloud ?
slug: utiliser-terraform
description: Utilisation de Terraform
keywords: infrastructure, instance, cloud, creation
excerpt: Décrouvez comment utiliser l'outil Terraform pour abstraire le déploiement de votre infrastructure
section: Tutoriels
---

**Dernières mise à jour le 09/06/2021**

## Objectif

OpenStack est un système d'exploitation de Cloud open source pour la création et la gestion de plateformes de cloud computing publiques et privées. Les composants logiciels OpenStack constituent la base de l'infrastructure Public Cloud de OVHcloud.

L'outil Open Source Terraform a été développé pour faciliter la création d'infrastructures de Cloud complexes. Il permet d'extraire les propriétés de votre infrastructure dans des fichiers texte qui peuvent servir de base au déploiement de votre infrastructure.

**Découvrez comment utiliser Terraform sur le Public Cloud OVHcloud.**

## Prérequis

* [Configurer un accès utilisateur à Horizon](../creer-un-acces-a-horizon/)
* [Préparer l’environnement pour utiliser l’API OpenStack](../preparer-lenvironnement-pour-utiliser-lapi-openstack/)
* [Charger les variables d'environnement OpenStack](../charger-les-variables-denvironnement-openstack/)
* [Disposer de vos identifiants API et de votre clé d'autorisation OVHcloud](https://docs.ovh.com/ca/fr/api/api-premiers-pas/)
* [Une clé SSH](../creation-des-cles-ssh/)
* [Package Terraform](https://www.terraform.io/intro/getting-started/install.html){.external}
* [Fichiers éxécutables OpenStack](https://access.redhat.com/documentation/en-us/){.external}

> [!primary]
>
> Ce tutorial a été réalisé avec la version suivante : Terraform v0.9.11
>

## En pratique

### Creer un environement Terraform

Après l'installation de Terraform, ipour tous les fichiers texte décrivant votre infrastructure :

```console
mkdir test_terraform && cd test_terraform
```

Vous pouvez à présent créer un environnement Terraformavec la commande suivante. Cela vous permettra de créer et de gérer l'évolution de votre infrastructure.

```console
terraform env new test_terraform
```

### Creer des ressources

#### Definir un fournisseur

Dans Terraform, vous spécifiez des « fournisseurs » (ou *providers*) pour votre environnement cloud. Un « fournisseur » (tel que OVHcloud) héberge vos ressources d'infrastructure OpenStack.

Dans un fichier nommé *provider.tf*, insérez les lignes suivantes :

Un fournisseur, comme OVHcloud, vous donne un environnement pour créer et développer des applications. Dans Terraform, un fournisseur est le point d'entrée de votre environnement de déploiement.

Dans un fichier provider.tf, ajoutez les lignes suivantes :

```python
# Configure le fournisseur OpenStack hébergé par OVHcloud
provider "openstack" {
  auth_url = "https://auth.cloud.ovh.net/v3" # URL d'authentification
  domain_name = "default" # Nom de domaine - Toujours à "default" pour OVHcloud
  alias = "ovh" # Un alias
}
```

Faites un clic-droit <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF1.txt" download>ici et cliquez sur « Enregistrer le lien sous »</a> pour télécharger uniquement le code ci-dessus en fichier texte.

Un « alias » est un identifiant unique pour un type de fournisseur. Par exemple, si vous avez deux fournisseurs OpenStack avec des informations d'identification différentes, vous devez spécifier chaque fournisseur dans la ressource.

#### Creer une instance

Dans Terraform, une « ressource » est un composant de votre infrastructure. Il peut s'agir d'une instance, d'un bloc de stockage, fourni par un fournisseur OpenStack ou un réseau fourni par le fournisseur OVHcloud.

Pour créer une instance simple, vous avez besoin de 4 éléments :

* Un nom d'instance
* Une image
* Un type d'instance (flavor)
* Une clé SSH

Pour lister les différents *flavors* (types d'instances) disponibles, vous pouvez entrer la commande suivante :

```console
nova flavor-list
```

Les exemples suivants utilisent la flavor « s1-2 ».

Pour lister les différentes images disponibles, vous pouvez entrer la commande suivante :

```console
glance image-list
```

Dans cette liste, nous décidons de prendre l'image « Ubuntu 16.04 ».

A des fins d'exemple, nous allons créer une instance simple sur Ubuntu 16.04 avec la flavor s1-2, et importer une clé SSH. Ajoutez les lignes suivantes dans un fichier nommé *simple_instance.tf* :

```python
1. # Création d'une ressource de paire de clés SSH
2. resource "openstack_compute_keypair_v2" "test_keypair" {
3.   provider = "openstack.ovh" # Nom du fournisseur déclaré dans provider.tf
4.   name = "test_keypair" # Nom de la clé SSH à utiliser pour la création
5.   public_key = "${file("~/.ssh/id_rsa.pub")}" # Chemin vers votre clé SSH précédemment générée
6. }
7. 
8. # Création d'une instance
9. resource "openstack_compute_instance_v2" "test_terraform_instance" {
10.   name = "terraform_instance" # Nom de l'instance
11.   provider = "openstack.ovh" # Nom du fournisseur
12.   image_name = "Ubuntu 16.04" # Nom de l'image
13.   flavor_name = "s1-2" # Nom du type d'instance
14.   # Nom de la ressource openstack_compute_keypair_v2 nommé test_keypair
15.   key_pair = "${openstack_compute_keypair_v2.test_keypair.name}"
16.   network {
17.     name = "Ext-Net" # Ajoute le composant réseau pour atteindre votre instance
18.   }
19. }
```

Faites un clic-droit <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF2.txt" download>ici et cliquez sur « Enregistrer le lien sous »</a> pour télécharger uniquement le code ci-dessus en fichier texte.

Tous les fichiers sont maintenant créés, vous pouvez entrer la commande suivante pour importer votre clé SSH et créer votre première instance :

```console
terraform apply
```

Si vous vous connectez à votre [espace client OVHcloud](https://ca.ovh.com/auth/?action=gotomanager&from=https://www.ovh.com/ca/fr/&ovhSubsidiary=qc), vous devriez y trouver une instance nommée « terraform_instance ».

Notez que la création d'une seconde instance identique avec « terraform apply » ne fonctionnera pas. Terraform applique les modifications uniquement s'il détecte une différence dans vos fichiers de configuration ou si un nouveau fichier est créé.

#### Créer des instances multiples

Dans cette partie, nous souhaitons créer une instance sous Ubuntu avec la flavor « s1-2 » dans chaque région.

Vous pouvez rechercher tous les noms de régions en utilisant cet appel API OVHcloud :

> [!api]
>
> @api {GET} cloud/project/{serviceName}/region
>

Nous utiliserons les régions OVHcloud suivantes pour cet exemple :

* GRA3
* SBG3
* BHS3

Vous pouvez créer trois ressources nommées « openstack_compute_instance_v2 » et modifier le paramètre de région pour chacune d'elles. Néanmoins, il peut devenir difficile de gérer des fichiers avec une grande quantité de ressources identiques.

C'est pour cela que Terraform propose un meta paramètre appelé « count ». Ce paramètre permet de préciser à Terraform de créer plusieurs fois une même ressource.

Pour ce faire, vous pouvez créer un fichier nommé « multiple_instance.tf ». Vous y définirez d'abord une variable contenant les trois régions, puis vous ajoutez un compteur de création d'instances :

```python
1. # Créer une variable région contenant la liste des régions d'OVHcloud
2. # On l'utilisera pour itérer sur les différentes régions afin de
3. # démarrer une instance sur chacune d'entre elles.
4. variable "region" {
5.   type = "list"
6.   default = ["GRA3", "SBG3", "BHS3"]
7. }
8. 
9. # Création d'une paire de clé SSH
10. resource "openstack_compute_keypair_v2" "test_keypair_all" {
11.   count = "${length(var.region)}"
12.   provider = "openstack.ovh" # Préciser le nom du fournisseur
13.   name = "test_keypair_all" # Nom de la clé SSH
14.   public_key = "${file("~/.ssh/id_rsa.pub")}" # Chemin de votre clé SSH
15.   region = "${element(var.region, count.index)}"
16. }
17. 
18. # Créer une ressource qui est une instance OpenStack dans chaque région
19. resource "openstack_compute_instance_v2" "instances_on_all_regions" {
20.   # Nombre de fois où la ressource sera créée
21.   # défini par la longueur de la liste nommée région
22.   count = "${length(var.region)}"
23.   provider = "openstack.ovh" # Nom du fournisseur
24.   name = "terraform_instances" # Nom de l'instance
25.   flavor_name = "s1-2" # Nom du type d'instance
26.   image_name = "Ubuntu 16.04" # Nom de l'image
27.   # element est une fonction qui accède à l'élément à la position
28.   # count.index de la liste var.region. Il permet d'itérer entre les régions
29.   region = "${element(var.region, count.index)}"
30.   # Accède au nom de la variable de la ressource openstack_compute_keypair_v2 nommée test_keypair
31.   key_pair = "${element(openstack_compute_keypair_v2.test_keypair_all.*.name, count.index)}"
32.   network {
33.     name = "Ext-Net" # Ajoute le réseau public à votre instance
34.   }
35. }
```

Faites un clic-droit <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF3.txt" download>ici et cliquez sur « Enregistrer le lien sous »</a> pour télécharger uniquement le code ci-dessus en fichier texte.

Appliquez vos modifications à l'aide de la commande suivante :

```console
terraform apply
```

Terraform peut créer plusieurs instances avec cette méthode, mais vous pouvez l'utiliser également pour modifier votre infrastructure actuelle.

#### Modifier une instance

Dans cet exemple, nous allons attacher un nouveau volume de stockage à notre première instance. Ouvrez et modifiez le fichier nommé « simple_instance.tf », puis ajoutez les lignes suivantes :


```python
1. # Créer une ressource de stockage
2. resource "openstack_blockstorage_volume_v2" "volume_to_add" {
3.   name = "simple_volume" # Nom du volume
4.   size = 10 # Taille du volume en GB
5.   provider = "openstack.ovh" # Nom du fournisseur
6. }
7. 
8. # Ajouter le volume créé précédemment à l'instance
9. resource "openstack_compute_volume_attach_v2" "attached" {
10.   # Identifiant de la ressource openstack_compute_instance_v2 nommée test_terraform_instance
11.   instance_id = "${openstack_compute_instance_v2.test_terraform_instance.id}"
12.   # Identifiant de la ressource openstack_blockstorage_volume_v2 nommée volume_to_add
13.   volume_id = "${openstack_blockstorage_volume_v2.volume_to_add.id}"
14. }
```

Faites un clic-droit <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF4.txt" download>ici et cliquez sur « Enregistrer le lien sous »</a> pour télécharger uniquement le code ci-dessus en fichier texte.

Appliquez vos modifications à l'aide de la commande suivante :

```console
terraform apply
```

#### Créer une instance dans le réseau OVHcloud (vRack)

Le plug-in Terraform OVHcloud peut gérer des réseaux privés, des sous-réseaux privés, des utilisateurs Public Cloud et des liaisons vRack. Dans cette partie, nous nous concentrerons sur la création du réseau.

Premièrement, vous devez charger les identifiants OVHcloud dans votre environnement. Personnalisez les commandes avec vos valeurs réelles :

```console
1. $ export OVH_ENDPOINT=ovh-eu
2. $ export OVH_APPLICATION_KEY=Votre_cle_dapplication_OVH(ou_AK)
3. $ export OVH_APPLICATION_SECRET=Votre_cle_dapplication_secrete_OVH(ou_AS)
4. $ export OVH_VRACK=Identifiant_vRack_ovh
5. $ export OVH_PUBLIC_CLOUD=Votre_tenant_id_openstack
6. $ export TF_ACC=1
7. $ export OVH_CONSUMER_KEY=Votre_token(ou_CK)
```

Ajoutez ensuite au fichier « provider.tf » les lignes suivantes :

```python
1. # Configuration du fournisseur
2. provider "ovh" {
3.   endpoint = "ovh-eu" # Point d'entrée du fournisseur
4.   alias = "ovh" # Alias du fournisseur
5. }
```

Faites un clic-droit <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF5.txt" download>ici et cliquez sur « Enregistrer le lien sous »</a> pour télécharger uniquement le code ci-dessus en fichier texte.

Créez à présent un fichier « create_private_network_instance.tf » et insérez-y le contenu suivant :

```python
1. # Création d'un réseau privé
2. resource "ovh_publiccloud_private_network" "network" {
3.    project_id = "OS_TENANT_ID" # Remplacez OS_TENANT_ID par votre Tenant ID de projet
4.    name = "private_network" # Nom du réseau
5.    regions = ["OS_REGION_NAME"] # Remplacez OS_REGION_NAME par la variable d'environnement OS_REGION_NAME
6.    provider = "ovh.ovh" # Nom du fournisseur
7.    vlan_id = 168 # Identifiant du vlan pour le vRrack
8. }
9. 
10. # Création d'un sous-réseau grâce au réseau privé créé précédemment
11. resource "ovh_publiccloud_private_network_subnet" "subnet" {
12.    project_id = "OS_TENANT_ID" # Remplacez OS_TENANT_ID par votre Tenant ID de projet
13.    # Identifiant de la ressource ovh_publiccloud_private_network nommée network
14.    network_id = "${ovh_publiccloud_private_network.network.id}"
15.    start = "192.168.168.100" # Première IP du sous réseau
16.    end = "192.168.168.200" # Dernière IP du sous réseau
17.    network = "192.168.168.0/24" # Place d'adressage IP du sous réseau
18.    dhcp = true # Activation du DHCP
19.    region = "OS_REGION_NAME" # Remplacez OS_REGION_NAME par la variable d'environnement OS_REGION_NAME
20.    provider = "ovh.ovh" # Nom du fournisseur
21. }
22. 
23. # Création d'une instance avec 2 interfaces réseau
24. resource "openstack_compute_instance_v2" "proxy_instance" {
25.   provider = "openstack.ovh" # Nom du fournisseur
26.   name = "proxy_instance" # Nom de l'instance
27.   image_name = "Ubuntu 16.04" # Nom de l'image
28.   flavor_name = "s1-2" # Nom du type d'instance
29.   # Nom de la ressource openstack_compute_keypair_v2 nommée test_keypair
30.   key_pair = "${openstack_compute_keypair_v2.test_keypair.name}"
31.   # Ajout du réseau public et privé
32.   network = [{name = "Ext-Net"}, {name = "${ovh_publiccloud_private_network.network.name}"}]
33. }
```

Faites un clic-droit <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF6.txt" download>ici et cliquez sur « Enregistrer le lien sous »</a> pour télécharger uniquement le code ci-dessus en fichier texte.

Appliquez vos modifications à l'aide de la commande suivante :

```console
terraform apply
```

Vous verrez apparaître dans votre projet Public Cloud une nouvelle instance, avec une interface publique et une interface privée.

#### Creation d'une infrastructure pour un site web

Dans cet exemple, nous allons créer une infrastructure de site Web de base à l'aide de Terraform et du réseau privé OVHcloud. Les composants créés sont les suivants :

* Un réseau privé
* Un sous-réseau
* 2 instances avec 2 interfaces réseau chacune : la première publique et la seconde privée, représentant les frontaux web
* 1 instance avec 1 interface réseau et 2 disques additionnels, représentant la BDD

![public-cloud](images/basic_infrastructure_for_a_web_site.png){.thumbnail}

Créez un fichier « simple_web_site.tf » et entrez les lignes suivantes :

```python
1. # Création d'un réseau privé
2. resource "ovh_publiccloud_private_network" "private_network" {
3.   name = "backend" # Nom du réseau
4.   regions = ["OS_REGION_NAME"] # Remplacez OS_REGION_NAME par la variable d'environnement OS_REGION_NAME
5.   provider = "ovh.ovh" # Nom du fournisseur
6.   project_id = "OS_TENANT_ID" # Remplacer OS_TENANT_ID par votre Tenant ID de projet
7.   vlan_id = 42 # Identifiant du vlan pour le vRrack
8. }
9. 
10. # Création d'un sous réseau privé
11. resource "ovh_publiccloud_private_network_subnet" "private_subnet" {
12.   # Identifiant de la ressource ovh_publiccloud_private_network nommée private_network
13.   network_id = "${ovh_publiccloud_private_network.private_network.id}"
14.   project_id = "OS_TENANT_ID" # Remplacez OS_TENANT_ID par votre Tenant ID de projet
15.   region = "OS_REGION_NAME" # Remplacez OS_REGION_NAME par la variable d'environnement OS_REGION_NAME
16.   network = "192.168.42.0/24" # IP du sous réseau
17.   start = "192.168.42.2" # Première IP du sous réseau
18.   end = "192.168.42.200" # Dernière IP du sous réseau
19.   dhcp = false # Désactivation du DHCP
20.   provider = "ovh.ovh" # Nom du fournisseur
21. }
22. 
23. # Chercher l'image Archlinux la plus récente
24. data "openstack_images_image_v2" "archlinux" {
25.   name = "Archlinux" # Nom de l'image
26.   most_recent = true # Limite la recherche à la plus récente
27.   provider = "openstack.ovh" # Nom du fournisseur
28. }
29. 
30. # Liste d'adresses IP privées possibles pour les frontaux
31. variable "front_private_ip" {
32.   type = "list"
33.   default = ["192.168.42.2", "192.168.42.3"]
34. }
35. 
36. # Création de 2 instances avec 2 interfaces réseau
37. resource "openstack_compute_instance_v2" "front" {
38.   count = "${length(var.front_private_ip)}" # Nombre d'instances à créer
39.   provider = "openstack.ovh" # Nom du fournisseur
40.   name = "front" # Nom de l'instance
41.   key_pair = "${openstack_compute_keypair_v2.test_keypair.name}"
42.   flavor_name = "s1-2" # Nom du type d'instance
43.   image_id = "${data.openstack_images_image_v2.archlinux.id}" # Identifiant de l'image de l'instance
44.   security_groups = ["default"] # Ajoute l'instance au groupe de sécurité
45.   network = [
46.     {
47.       name = "Ext-Net" # Nom de l'interface réseau publique
48.     },
49.     {
50.       # Nom de l'interface réseau privé
51.       name = "${ovh_publiccloud_private_network.private_network.name}"
52.       # Adresse IP prise depuis la liste définie précédemment
53.       fixed_ip_v4 = "${element(var.front_private_ip, count.index)}"
54.     }
55.   ]
56. }
57. 
58. # Création d'un périphérique de stockage attachable pour le backup (volume)
59. resource "openstack_blockstorage_volume_v2" "backup" {
60.   name = "backup_disk" # Nom du périphérique de stockage
61.   size = 10 # Taille
62.   provider = "openstack.ovh" # Nom du fournisseur
63. }
64. 
65. # Création d'une instance avec une interface réseau et d'un phériphérique de stockage
66. resource "openstack_compute_instance_v2" "back" {
67.   provider = "openstack.ovh" # Nom du fournisseur
68.   name = "back" # Nom de l'instance
69.   key_pair = "${openstack_compute_keypair_v2.test_keypair.name}"
70.   flavor_name = "s1-2" # Nom du type d'instance
71.   image_id = "${data.openstack_images_image_v2.archlinux.id}" # Identifiant de l'image de l'instance
72.   security_groups = ["default"] # Ajoute l'instance au groupe de sécurité
73.   network = [
74.     {
75.       name = "${ovh_publiccloud_private_network.private_network.name}" # Nom du réseau privé
76.       fixed_ip_v4 = "192.168.42.150" # Addresse IP privée choisie arbitrairement
77.     }
78.   ]
79.   # Périphérique de stockage bootable contenant l'OS
80.   block_device {
81.     uuid = "${data.openstack_images_image_v2.archlinux.id}" # Identifiant de l'image de l'instance
82.     source_type = "image" # Type de source
83.     destination_type = "local" # Destination
84.     volume_size = 10 # Taille
85.     boot_index = 0 # Ordre de boot
86.     delete_on_termination = true # Le périphérique sera supprimé quand l'instance sera supprimée
87.   }
88.   # Périphérique de stockage
89.   block_device {
90.     source_type = "blank" # Type de source
91.     destination_type = "volume" # Destination
92.     volume_size = 20 # Taille
93.     boot_index = 1 # Ordre de boot
94.     delete_on_termination = true # Le périphérique sera supprimé quand l'instance sera supprimée
95.   }
96.  # Périphérique de stockage créé précédemment
97.  block_device {
98.    uuid = "${openstack_blockstorage_volume_v2.backup.id}" # Identifiant du périphérique de stockage
99.    source_type = "volume" # Type de source
100.    destination_type = "volume" # Destination
101.    boot_index = 2 # Ordre de boot
102.    delete_on_termination = true # Le périphérique sera supprimé quand l'instance sera supprimée
103.  }
104. }
```

Faites un clic-droit <a href="https://raw.githubusercontent.com/ovh/docs/develop/pages/platform/public-cloud/how_to_use_terraform/images/TF7.txt" download>ici et cliquez sur « Enregistrer le lien sous »</a> pour télécharger uniquement le code ci-dessus en fichier texte.

Appliquez vos modifications à l'aide de la commande suivante :

```console
terraform apply
```

### Supprimer une infrastructure

Pour supprimer toutes les ressources, vous pouvez entrer la commande suivante:

```console
terraform destroy
```

## Aller plus loin

Rejoignez notre communauté d'utilisateurs sur <https://community.ovh.com/>.
