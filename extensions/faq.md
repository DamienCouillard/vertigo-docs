# FAQ

Nous présentons ici, les éléments questions les courantes.
N'hésitez pas à nous contacter à support@vertigo.io ou sur notre discord 

## Mon projet stocke des pièces jointes, quel type de stockage choisir ?
Vertigo propose plusieurs type de stockage.
Le choix dépendra de la volumétrie et des contraintes de l'hébergeur.
Si le volume est faible, un stockage en base de données est possible.
Sinon, préférer un stockage metadonnée en base de données et fichier sur FileSystem
En cas de gros volume, un stockage objet (type min.io) peut être préférable

## Les composants ne semblent pas fonctionner
Pour activer les composants, il faut le faire dans le config de SpringMvc : le fichier de config de ton projet doit hériter du VSpringWebConfig de VertigoUi , il met toute la conf Spring nécessaire.
Regarde l'exemple de la config de Mars : https://github.com/vertigo-io/vertigo-mars/blob/master/src/main/java/io/mars/support/boot/MarsVSpringWebConfig.java
Normalement l'archetype Maven le pose déjà comme il faut.

## La page refuse de s'afficher
Si la page contient layout:decorate="~{templates/MonLayout}", alors il faut que la page respecte la structure de MonLayout
Un layout c'est la page complete avec des trous
Pour faire une page on indique quel layout prendre et ce que l'on met dans les trous.
Il est possible de faire plusieurs niveau de layout mais ca n'aide pas la lisibilité alors il n'en faut pas trop
En principe, un layout général, un layout pour les pages de recherche, d'accueil ou autre, un layout pour les pages de détail

## Quel outil de modelisation de données utiliser ?
Vertigo studio est nativement compatible avec PowerDesigner et Entreprise Architect.
PowerDesigner est préconisé car plus complete, Entreprise Architect passe par le XMI

## Ou sont les classes css du genre : col-md-3 col-xs-12 q-jumbotron bg-white
Ce sont des classes fournient par la librairie de composant Quasar (https://quasar.dev/layout/grid/introduction-to-flexbox#Responsive-Design)

## Comment debuger les écrans vue.js/quasar ?
Il existe une extension navigateur pour vueJs qui aide au debug : `Vue.js devtools`. Pour l'utiliser il faut vue.js en version non minifiée (à ajouter au début de la page)
Sinon la vue developpeur et le débug peuvent être utilisée.

## Les boutons `<vu:button-link>` ne fonctionnent que si ils sont placés à l'interieur de balises `<section>`
La balise section est liée aux layout thymeleaf.
Tous codes html hors des balises qui sont effectivement inclus dans la page sont gardés, le reste est perdu.

## Coté IHM comment accéder dans la page aux données mises dans le context ?
Les données coté client sont accessible dans le VUiPage.vueData
Seule les données demandées lors du rendu coté serveur sont accessibles coté client afin d'améliorer la sécurité de l'application
Le mieux est tout de meme de privilégier au maximum le rendu coté serveur
Ainsi pour afficher des informations statiques le mieux est de le faire avec une balise thymeleaf directement coté serveur

## Comment mettre une liste de référence dans le context ?
Une liste de référence est ajoutée dans le context avec la méthode `publishMdl`
Il faut au préalable déclarer la liste de référence :
Il faut un DefinitionProvider, par exemple : `MarsMasterDataDefinitionProvider`
Le définitionProvider doit être ajouté dans la configuration du module, par exemple `io.mars.support.SupportFeatures`
StaticMasterData permettent d'avoir des enums pour les listes de référence statique donc non administrables via ihm

## A quoi sert le tag `<vu:include-data>` dans certains écrans de la démo mars ?
Ce tag permet d'inclure la donnée du context serveur dans le vueData client.
Normalement le composant d'affichage s'occupe du include-data et il n'y a rien à faire.
Dans certains cas, il n'y a pas de composant d'affichage (ni vu:textfield, ni vu:column, ...) mais on en a besoin coté client (pour construire un lien par exemple), il faut alors l'inclure manuellement.

## J'ai un <vu:select> dans mon formulaire, il permet d'afficher le libellé et non l'id, comment reproduire le comportement dans une liste avec le <vu:column> ?
Dans de nombreux cas, l'objet sous-jacent à une liste est un objet sépcifique d'IHM, il est alors possible d'ajouter un champ dans la liste, et adapter le select SQL pour récupérer le libellé directement.
Dans le cas d'une liste de référence (sinon attention aux performances), cela peut-être fait automcatiquement en définissant le contenu de la colonne :
```
<vu:column name="equipmentType" label="Equipment Type" >
    <vu:field-read field="equipmentTypeId" list="equipmentTypes" listKey="equipmentTypeId" listDisplay="label" />
</vu:column>
```
Il y a deux manière de définir une colonne :
- en référencant un field
- en définissant un name puis le contenu de la colonne
Une fois que tu es dans le cas deux tu peux utiliser comme contenu un champ spécial `<vu:field-read>` qui s'occupe d'afficher un champ en read-only qui pointe vers une liste

## Comment rendre les éléments d'une liste sélectionnable ?
Il suffit de poser l'attribut selectable="true" sur la table.
Cela active un binding de la selection dans un : componentStates.${componentId}.selected 
Pour emettre la selection coté serveur, il faut ajouter du code spécifique.

## Comment rendre un champ obligatoire en fonction d'un autre ?
Il faut utiliser un DtObjectValidator
Voici le code à mettre dans la methode de controleur pour lancer le controle du validateur sur l'objet
```
viewContext.getUiObject(contextKey).mergeAndCheckInput(Collections.singletonList(new YourCustomDtObjectValidator()), uiMessageStack);
if (uiMessageStack.hasErrors()) {
            throw new ValidationUserException();
 }
```
Pour récuper l'uiMessageStack il suffit de l'inclure dans la signature de la méthode du controlleur (comme le ViewContext)

## Comment envoyer un mail ?
Le MailManager aide pour l'envoi de mail. (https://github.com/vertigo-io/vertigo-extensions/blob/master/vertigo-social/src/test/java/io/vertigo/social/mail/MailManagerTest.java)


## Peux-t-on avoir 2 balises <vu:messages> dans une page ?
Non il faut une seule
Le plus simple est que le <vu:message> soit dans le template parent

## L'implémentation d'un Manager vertigo est introuvable (Components or params not found)
Il faut pensser à activer la fonctionnalité dans le fichier de configuration yaml de l'appli (https://vertigo-io.github.io/vertigo-docs/#/basic/configuration)

## Comment rendre un paramètre du fichier de configuration yaml modifiable par l'hébergeur ?
Les paramètres peuvent être extarnalisés avec une balise de type : ${myParamName}
La valeur est alors résolue par le paramManager.

## [Studio] J'essaye de faire une double asscociation dans un .ksp vers le même DtObject, mais studio génère deux méthodes avec le même nom.
Il faut donner un rôle à chaque association (roleA et roleB), ce rôle est utilisé pour nommer la méthode de navigation


## Je souhaite faire apparaitre une notification à l'utilisateur, comment faire ?
Il faut utiliser l'api Notify de Quasar (cf : https://quasar.dev/quasar-plugins/notify#Notify-API)
Attention, si c'est après un appel Ajax, pour récupérer le `$q`de Quasar, il faut :
- soit binder la fonction sur this pour pouvoir utiliser `$`q : `function(response){ this.$q.notify({message : 'TEST', type : 'positive'}).bind(this)`
- soit passer par l'instance Globale de `VUiPage.$q.notify`


## Quel est l'api pour faire des appels Ajax ?
La signature de la méthode `httpPostAjax` est la suivante :
```httpPostAjax(url, params, options)```

Le derniere paramètre permet de fournir un objet qui contient le callback en cas de succès et en cas d'erreur
```{
    onSuccess : function (response) {
        // do something
    }, 
    onError(error) {
        // do something
    }
}```

## Comment proposer à un utiliseteur de selectionner plusieurs choix parmi les éléments d'une liste de référence  ?
Cela dépend du mode de stockage.
Mais globalement, nous proposons deux fonctionnements : 
1- Dans l'objet de critère, on ajoute un champ avec le domain de la FK et une cardinalité `*`
Dans ce cas le champ sera bien ajouté au vueData en tant que tableau d'id et pourra être mappé sur la checkbox comme ceci :
```
<q-checkbox v-model="selectedTimeZoneList" v-for="item in vueData.timeZoneList" :val="item" :label="item"></q-checkbox>
``` 
Coté `Controller`, il y aura un champ qui est un tableau d'id. Ce champ n'est pas persistable en tant que tel, charge au service de le traduire en donnée persistable.

2- L'autre solution, constiste à utiliser les *SmartTypes*. 
Dans l'objet critère, on ajoute un champ avec un domain qui est un SmartType (par exemple `DoIds`) avec comme BasicType une String
On associe au SmartType un adapteur UI qui transforme la chaine de caractère une liste d'Id, celle-ci sera sérialisée en Json lors de l'ajout au vueData, et pourra être utilisé par le composant checkbox comme dans le cas 1.
Coté `Controller`, le champ sera une chaine de caractère qui pourra être persistée directement si besoin.


## Je n'arrive pas à faire fonctionner l'upload de fichier en Ajax

Coté page :
```Html
<vu:fileupload th:if="${model.modeEdit}" float-label="Add new pictures here" th:url="'@{/commons/upload}'" key="baseTmpPictureUris" multiple />	
```

Coté Controller
```Java
@PostMapping("/_save")
	public String doSave(
			final ViewContext viewContext,
			@Validate(DefaultDtObjectValidator.class) @ViewAttribute("base") final Base base,
			@QueryParam("baseTmpPictureUris") final List<FileInfoURI> addedPictureFile,
			final UiMessageStack uiMessageStack) {
```

Le composant d'upload marche en deux temps : 
1- l'utilisateur dépose son fichier sur le composant, celui-ci envoi tout de suite le fichier coté serveur et récupère un id temporaire qu'il stock dans le formulaire
2- Lorsque l'utilisateur poste son formulaire, l'id du fichier part avec le reste des données métiers, coté serveur l'id permet de retrouver le fichier et la méthode du controlleur reçoit les données métiers et le fichier en entrée.

Lorsque l'étape 2 est faite en Ajax, il faut récupérer l'id *à la main*. Le code à ajouter ressemble à :
```Json
<q-btn th:@click="|httpPostAjax('', {baseTmpPictureUris:VUiPage.componentStates.uploaderbaseTmpPictureUris.fileUris.toString()})|"  label="Save"></q-btn>
```

## Comment choisir mon plugin de stockage de fichier ?
Vertigo propose plusieurs type de stockage.
Le choix dépendra de la volumétrie et des contraintes de l'hébergeur.
Si le volume est faible, un stockage en base de données est possible (avec `DbFileStorePlugin`).
Il faut alors un objet de mapping avec un champ `FILE_DATA` de type Blob (ou `bytea` sur PostgreSQL)

Sinon, préférer un stockage metadonnée en base de données et fichier sur FileSystem (avec `FsFileStorePlugin`).
Il faut alors un objet de mapping avec un champ `FILE_PATH` de type String dans lequel on stocke le path vers le fichier physique. 
Ce path doit pointer vers un espace adapté au context du projet (par exemple un NAS)


8/09








