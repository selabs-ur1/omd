# Software Development Industrialization

Introduction à la gestion de versions, la gestion de dépendances et l'intégration continue

## Objectifs du TP

- Comprendre le fonctionnement de maven, configurer un projet de développement, utiliser les artifacts, et générer des rapports
- Utiliser Git pour sauvegarder et collaborer sur le code source de votre projet, au travers des forges telles que Github et Gitlab
- Utiliser un système d’intégration continue tel que Sonar, Jenkins et Gitlab CI

## Partie 1 : Utilisation de Maven

### Liens utiles

- Site de Maven : http://maven.apache.org/
- FAQ MAVEN developpez.com : http://java.developpez.com/faq/maven/

### Environnement 

Selon le 3ième lien donnée ci-dessus, Maven est essentiellement un outil de gestion et de compréhension de projet. Maven offre des fonctionnalités de : construction, compilation ; documentation ; rapport ; gestion des dépendances ; gestion des sources ; mise à jour de projet ; déploiement.

Utiliser Maven consiste à définir dans chaque projet à gérer un script Maven appelés POM : *pom.xml*. Nous allons voir dans ce TP qu'un POM permet de définir des dépendances, des configurations pour notamment construire, tester, mettre en paquet des artefacts logiciels (exécutables, tests, documentations, archives, etc.). Pour cela, Maven récupère sur des dépôts maven les outils dont il a besoin pour exécuter le POM. Utiliser Maven requière donc : une (bonne) connexion à Internet car il télécharge beaucoup de choses ; de l'espace disque pour la même raison. Les artefacts qu'il télécharge sont originellement stockés dans le dossier *.m2* dans votre home-dir. Ce dossier contient également le fichier de configuration Maven : settings.xml.

Pour configurer Maven de manière à changer l'endroit où les artefacts téléchargés seront stockés (e.g., afin d'éviter des problèmes d'espace disque), vous pouvez modifier le fichier settings.xml de la manière suivante :

    <?xml version="1.0" encoding="UTF-8"?>
    <settings>
        <localRepository>/tmp/mavenrepository</localRepository>
        <offline>false</offline>
    </settings>

### Création d'un projet Maven

Création d’une application basique : pour initialiser un projet Java, vous pouvez utiliser l’archetype maven *maven-archetype-quickstart*. Vous avez juste à fournir un *groupId* et un *artefactId*.

Dans Eclipse:

    new -> other -> maven -> maven project. Vous devrez sélectionner l’archetype,  l’artifactId et le groupId

En ligne de commande (non nécessaire si vous l’avez fait depuis Eclipse):

    mvn archetype:create \
        -DgroupId=[your project's group id] \
        -DartifactId=[your project's artifact id] \
        -DarchetypeArtifactId=maven-archetype-quickstart

Ou simplement :

    mvn archetype:create \
        -DgroupId=[your project's group id] \
        -DartifactId=[your project's artifact id]

Vous obtenez la structure de projet jointe

    |-- src
    |   |-- main
    |   |   `-- java
    |   |       `-- [your project's package]   
    |   |           `-- App.java
    |   `-- test
    |       `-- java
    |           `-- [your project's package]   
    |               `-- AppTest.java
    `-- pom.xml

Par exemple si vous exécutez la commande
    
    mvn archetype:generate \
        -DgroupId=fr.esir.mdi.ci \
        -DartifactId=tpmaven

Vous obtiendrez l’architecture suivante :
 
    |-- src
    |   |-- main
    |   |   `-- java
    |   |       `-- fr
    |   |           `-- esir
    |   |               `-- mdi
    |   |                   `-- ci
    |   |                       `-- tpmaven
    |   |                           `-- App.java
    |   `-- test
    |       `-- java
    |           `-- fr
    |               `-- esir
    | 	                `-- mdi
    |		            `--ci
    |                           `-- tpmaven
    |                               `-- AppTest.java
    `-- pom.xml

Le fichier pom.xml est le fichier de configuration maven du projet. Il décrit les caractéristiques du projet (son nom, sa famille, sa version, etc.), ainsi que les processus (les « builds ») à exécuter (la compilation, l'exécution des tests, la création d'archive, etc.).

Il existe différentes tâches Maven de base, i.e. fournies par Maven. Les principales sont :
- mvn clean : supprimer le dossier target. Le dossier target d'un projet maven contient toutes les données produites par maven (classes compilées, jar produits, rapports, etc.) ;
- mvn compile: lance la compilation du code source du projet Maven (mais pas la compilation des tests) ;
- mvn test: mvn compile + lance la compilation et l'exécution des tests ;
- mvn package: mvn test + lance les opérations de packaging (exemple : la création de fichiers jar) ;
- mvn install: mvn package + installe les jar produits dans le dossier .m2 de l'utilisateur pour une utilisation dans les autres projets en local.

Chaque plug-in configuré et utilisé dans un pom peuvent fournir des tâches spécifiques.

### Configuration d'un projet Maven dans Eclipse

Depuis Eclipse 4.X, le support de maven s’est amélioré. Pour importer votre projet (ne pas faire si vous avez créé votre projet maven depuis eclipse, ce sera par contre à faire après avoir récupéré un projet depuis un serveur git) : 

    File -> import -> maven -> existing maven project.

Votre projet est configuré.

## Partie 2 : Gestion des dépendances

Intégrer à votre projet le fichier [ici](https://gist.github.com/combemale/54a8f2d29aba087627e6bf73eba3baa3).

Vous verrez que le code ne compile pas car il manque une dépendances. Intégrez maintenant la dépendance (_\<dependencies>...\<dependencies>_) à itext dans le fichier _pom.xml_.

    <!-- https://mvnrepository.com/artifact/com.itextpdf/itextpdf -->
    <dependency>
        <groupId>com.itextpdf</groupId>
        <artifactId>itextpdf</artifactId>
        <version>5.5.13.1</version>
    </dependency>

Votre IDE va downloader la dépendance et la mettre automatiquement dans votre classpath. Dans ce sens, cela permet de ne mettre dans votre gestionnaire de source que le code source et le descripteur de projet (pom.xml). 


## Partie 3 : Spécialisation du processus de build

Imaginons que vous souhaitiez ajouter une tâche dans le processus de build. Par exemple, compilez votre code source avec la version Java 11. Ajoutez la section suivant à votre fichier pom.xml (essayez en changant de version) :

    <build>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
              <!-- or whatever version you use -->
              <source>11</source>
              <target>11</target>
            </configuration>
          </plugin>
        </plugins>
    </build>

Vous pouvez ajouter de nombreux plugin dans cette section. Prenez le temps d'aller regarder [ici](https://maven.apache.org/plugins/).

## Partie 4 : Génération de rapports

### Générer la Javadoc

Ajoutez des commentaires à votre code, puis ajoutez le code suivant dans la section _build_ du pom.xml de votre projet.

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-site-plugin</artifactId>
        <version>3.9.1</version>
    </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-project-info-reports-plugin</artifactId>
        <version>3.1.1</version>
    </plugin>

Puis dans la section _reporting_:

    <reporting>
      <plugins>
       <plugin>
    	<groupId>org.apache.maven.plugins</groupId>
    	<artifactId>maven-javadoc-plugin</artifactId>
    	<version>3.2.0</version>
       </plugin>
     </plugins>
    </reporting>

Puis lancez en ligne de commande (au même niveau que le fichier *pom.xml*) : _mvn site_. Cette tâche crée un site Web pour votre projet. Par défaut, les goals maven générant des fichiers travaillent dans le dossier target se trouvant au même niveau que le fichier *pom.xml*. Allez dans le dossier target/site et ouvrez le fichier *index.html*. Vous pouvez regarder la Javadoc générée en cliquant sur Project reports.

Eclipse permet de lancer cette commande maven sans passer par la ligne de commande (voir menu run d'Eclipse). 

### Valider la qualité du code avec le plugin checkstyle

Ajoutez à la section \<plugins> dans \<reporting> le plugin checkstyle :

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-checkstyle-plugin</artifactId>
	    <version>3.1.1</version>
    </plugin>

Lancez *mvn clean site* (le goal clean vide le dossier target). Sur la page Web générée par « mvn site », une nouvelle section Checkstyle a été ajouté dans Project reports.

Quelle est la norme de codage à laquelle se réfère le rapport par défaut ? Comment imposer la norme de codage de Google? Le fichier de configuration de google est inclus dans checkstyle vous devez juste indiquer dans la configuration que vous souhaitez l’utiliser (cf. https://maven.apache.org/plugins/maven-checkstyle-plugin/examples/custom-checker-config.html). Modifiez votre classe du projet tp1 de façon à diminuer le nnombre d'erreurs. 

- Site de l'outil CheckStyle : http://checkstyle.sourceforge.net/
- Site du plugin Maven : http://maven.apache.org/plugins/maven-checkstyle-plugin/

### Rapport croisé de source

    <reporting>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jxr-plugin</artifactId>  
				<version>3.0.0</version>
			</plugin>
		</plugins>
    </reporting>

Lien utile : http://maven.apache.org/plugins/maven-jxr-plugin/
Quelle est la valeur ajoutée de ce plugin ? En particulier, montrez sa complémentarité avec CheckStyle.
Désormais vous pouvez passer du rapport CheckStyle au code source en cliquant sur le numéro de ligne associé au commentaire CheckStyle.

### Couverture des tests

A quel point les développeurs ont réalisé des tests unitaires ? Quelles parties de l'application n'ont pas été testées ?
Ecrivez quelques tests en Junit (_/tpmaven/src/test/java/fr/esir/mdi/ci/tpmaven/FirstPdfTest.java_), et voyez qu'elle couverture de code vous obtenez. 

	<build>
		<plugins>
			<plugin>
				<groupId>org.jacoco</groupId>
				<artifactId>jacoco-maven-plugin</artifactId>
				<version>0.8.6</version>
				<executions>
					<execution>
						<goals>
							<goal>prepare-agent</goal>
						</goals>
					</execution> <!-- attached to Maven test phase -->
					<execution>
						<id>report</id>
						<phase>prepare-package</phase>
						<goals>
							<goal>report</goal>
						</goals>
						<configuration>
							<outputDirectory>target/jacoco-report</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
    
	<reporting>
		<plugins>
			<plugin>
				<groupId>org.jacoco</groupId>
				<artifactId>jacoco-maven-plugin</artifactId>
				<reportSets>
					<reportSet>
						<reports>
							<!-- select non-aggregate reports -->
							<report>report</report>
						</reports>
					</reportSet>
				</reportSets>
			</plugin>
		</plugins>
	</reporting>

Lien utile : https://www.eclemma.org/jacoco/

### Identifier des patterns d'erreur avec PMD

Ajoutez volontairement du code mort à votre code (e.g., une méthode non utilisée) et identifiez le code mort (e.g., variables ou paramètres non utilisés) et la duplication de code (e.g., code copié/collé = possible bug copié/collé, code 'compliqué', e.g., trop de if...else ...).

Ajoutez à la section \<reporting> le plugin PMD :

    <project>
        <reporting>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-pmd-plugin</artifactId>
                    <version>3.14.0</version>
                </plugin>
            </plugins>
        </reporting>
    </project>

Quels sont les deux nouveaux rapports générés ? Qu'est ce que le rapport 'CPD Report' ? Qu'est ce que le rapport 'PMD Report' ?

### Connaître l'activité du projet

Combien et quels fichiers ont été modifiés par un développeur ? Commiter votre projet sur github ou sur le gitlab de l’istic. 

Ajoutez à la section \<reporting> le plugin changelog, et définissez la section \<scm> en fonction de votre application et gestionnaire de version :
    
    <project>
        <reporting>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-changelog-plugin</artifactId> 
                </plugin>
            </plugins>
        </reporting>

        <scm>
            <connection>scm:git:git://IPSERVEUR/repository1/monappli</connection>
            <url>http://IPSERVEUR/svn/monappli-web</url>
        </scm>
    </project>

ipserveur doit être remplacé par l’IP de votre repo svn ou git. 

Lancez _mvn site_. Que s'est t'il passé ?

Le répertoire /target/site situé dans votre projet contient maintenant trois rapports d'activité :
- changelog : rapport indiquant toutes les activités sur le SCM.
- dev-activity : rapport indiquant par développeur le nombre de commits, de fichiers modifiés.
- file-activity : rapport indiquant les fichiers qui ont été révisés.

## Partie 5 : Utilisation de Git

Pour cette partie, vous pouvez utiliser à votre convenance soit [la forge Gitlab](https://gitlab.istic.univ-rennes1.fr/) fournie par la plate-forme ISTIC ESIR, soit [GitHub](https://github.com/). Mettez votre code sur github (ou gitlab):
- créez un nouveau repository via l'interface github/gitlab
- liez le dépôt local au distant : 
    
    git remote add origin https://github.com/login/nomRepo.git

- mettez votre code sur ce dépôt : 
    
    git push origin master (en cas de non-fast-forward : git pull origin master)

Initiez vous aux principales commandes Git :

- créez deux branches, passez de l'une a l'autre dans votre repo local (_git checkout_), et faite des modifications (dont certaines en conflit entre les deux branches) dans chacune d'entre elles (e.g., rajoutez des commentaires, des tests, etc.)
- mergez successivement les deux branches avec la branche master.  Le merge de la deuxème branche devrait soulever des conflits que vous devez résoudre manuellement. Résolvez le conflit en éditant le fichier en conflit et en enlevant manuellement les <<<<>>>> etc. Ces lignes représentent les lignes en conflits entre vos 2 versions (des deux branches).
- poussez ces modifications sur le repository. 

Si vous êtes sur Github, vous pouvez aussi vous initier à la création de _pull request_, à l'utilisation de _bots_, etc. 

Vous pouvez annuler tous les commits précédents (_git revert_ ...) avant de poursuivre le TP. 

Vous allez dans la suite étudier les outils d'intégration continue Sonar et Jenkins/GitLab CI. La différence entre ces deux outils est simple : Sonar est un outil d'assurance qualité tandis que Jenkins est un outil de « release engineering ». Les deux sont complémentaires.

## Partie 6 : Intégration avec l'outil Sonar

Téléchargez [Sonar](https://www.sonarqube.org/downloads/), de-compressez-le dans */tmp*, puis lancez Sonar : 
    
    sh ./bin/linux-x86-64/sonar.sh start

Au niveau du pom de votre projet, lancez la commande *mvn sonar:sonar* puis allez à l'adresse http://localhost:9000/. Loguez-vous avec le login admin et le mot-de-passe admin. Allez dans Quality Profiles et changer les règles de qualités utilisées puis relancer mvn sonar:sonar. Baladez-vous dans Sonar pour explorer ces différentes fonctionnalités. 

Vous pourrez ensuite arrêter sonar avec
    
    sh ./bin/linux-x86-64/sonar.sh stop

## Partie 7 : Intégration avec Jenkins

Sur http://jenkins-ci.org/, prenez la version LTS Java Web Archive (.war) pour la mettre dans /tmp. Il faut déplacer l'endroit où la configuration Jenkins sera stockée :

    export JENKINS_HOME=/tmp/.jenkins

Démarrez jenkins : 

    java -jar jenkins.war --httpPort=9900 
    
Allez dans votre navigateur : http://localhost:9900/.

Configurez Jenkins :
- Commencez par cliquer sur « enable auto refresh »
- Allez dans le menu « Jenkins → Manage Jenkins → Global Tool Configuration »
Cliquez sur « Add JDK ». Saisissez un nom quelconque permettant d'identifier la JDK. Cochez « I agree... ». Ignorez le message d'erreur « requires Oracle account »
- Cliquez sur « Add Maven ». Saisissez un nom quelconque permettant d'identifier cette version de Maven.
- Cliquez sur « Save ». Le but de ces configurations est de pouvoir installer, si on le souhaite, plusieurs Maven ou plusieurs JDK (par exemple, certains projets peuvent nécessiter Java 6 et d'autres Java 8).
- Installez le module git pour Jenkins : « Jenkins → Manage Plugins → Available → « GIT plugin » et « Maven integration plugin » → Download now and install after restart → Restart Jenkins ». Ces plugins peuvent être déjà installés.

Par défaut, jenkins ne contient pas le plugin pour gérer des repository Git, Il vous faut installer le plugin “Git Plugin”. De plus, vous devez configurer Maven (voir Configure System).

Ensuite créez un « job » en cliquant sur « create new job -> Maven Project ». Donnez un nom à votre projet. Définissez les sources en indiquant l'url du repository git que vous avez préalablement créer sur github (i.e. https://github.com/login/nomRepo.git) et enfin définissez les goals maven pour le build (« Add build step » → « Invoke top-level Maven... ») : pour commencer clean package. Si le pom n’est pas à la racine de votre projet, cliquez sur « Advanced... » → remplissez le champ POM. Lancer un build.

Dans l'historique des builds, une icône bleu doit apparaître à la fin de la construction pour désigner la construction correcte de l'artefact (bleu car le développeur de Jenkins est Japonais et au Japon le bleu équivaut au vert chez nous, d'ailleurs un plugin Jenkins existe pour afficher des icônes vertes et non bleues...). Cliquez ensuite sur le lien sous « Module builds », les artefacts créés par jenkins en utilisant le POM du projet sont visibles dont un jar. Ouvrez ce dernier, vous verrez que le manifest est vide. Dans les étapes suivantes vous allez compléter le POM pour obtenir un vrai jar exécutable.

### Packager des artefacts logiciels avec maven

Les artefacts logiciels peuvent être produits soit en utilisant maven en ligne de commande (ou au travers de l'IDE), soit en utilisant un outil d'intégration continue tel que Jenkins. Dans cette partie, nous allons étudier différents plugins maven permettant de réaliser des actions liées à la construction d'artefacts logiciels, et qu'il faudra automatiser à l'aide de jenkins. 

#### Création d'un jar exécutable via maven

Pour construire des artefacts vous allez ajouter un bloc \<build> dans le bloc \<project> de votre POM. Générez un jar exécutable grâce au plugin maven-jar-plugin qui vous permettra de définir un manifest :
http://maven.apache.org/plugins/maven-jar-plugin/ (regardez les exemples « creating an executable JAR file »).

Lancez _mvn clean install_ et exécutez le nouveau jar généré se trouvant dans le dossier target. Commitez et pushez vos changements, relancez le build Jenkins, allez dans le « last build » et cliquez sur le « Module Builds » listé : la liste des éléments produits doit être visible et téléchargeable.

#### Exécution de test via maven
Utilisez le plugin maven-surefire-plugin pour exécuter les tests du projet lors de la commande _mvn clean install_, cf.: http://maven.apache.org/surefire/maven-surefire-plugin/
Commitez le POM sur github (avec quelques tests) et relancez un build sur Jenkins afin d'observer les évolutions apportées.

#### Création d'archives des sources et des exécutables
Le plugin maven-assembly-plugin permet de créer des archives. Ce plugin est notamment très utile pour créer des archives des sources ou des fichiers exécutables, cf : http://maven.apache.org/plugins/maven-assembly-plugin/ (voir aussi: https://medium.com/@kasunpdh/using-the-maven-assembly-plugin-to-build-a-zip-distribution-5cbca2a3b052)

Étudiez et adaptez l'utilisation de ce plugin dans le projet suivant :
https://github.com/latexdraw/latexdraw/blob/master/pom.xml
pour l'utiliser dans votre projet afin de créer un zip des sources et un autre contenant le jar exécutable.

Commitez les modifications sur github et relancez un build sur Jenkins afin d'observer les évolutions apportées. 

### Utilisation de sonar, cobertura et pmd

- https://wiki.jenkins-ci.org/display/JENKINS/Sonar+plugin
- https://wiki.jenkins-ci.org/display/JENKINS/Cobertura+Plugin
- https://wiki.jenkins-ci.org/display/JENKINS/PMD+Plugin

Attention pour Cobertura, vous avez besoin de définir le format de sortie en xml.
Pour cela, il existe deux solutions:
- la première consiste à ajouter une option dans la définition du build maven: “-Dcobertura.report.format=xml”
- la deuxième consiste à modifier la configuration dans votre pom et d’ajouter l’option de configuration  appropriée (voir sur la page de Cobertura plugin)

### Daily/Nightly build avec Jenkins

Configurer vos builds Jenkins pour qu'ils se construisent automatiquement à 1h du matin tous les jours.

## Partie 8 : Intégration avec GitLab CI

Pour cette partie, votre projet devra hébergé sur GitLab (e.g., https://gitlab.istic.univ-rennes1.fr), avec la permission d'exécuter des pipelines (Settings / General / Permissions / Pipelines). 

Quelques définitions préliminaires des concepts de [GitLab CI](https://docs.gitlab.com/ee/ci/README.html) : 
- [pipeline](https://docs.gitlab.com/ee/ci/pipelines/) : un ensemble de _jobs_ (quoi faire?), chacun a réaliser lors d'un _stage_ (quand le faire?).
- [runner](https://docs.gitlab.com/ee/ci/runners/README.html) : GitLab utilise des runners sur différents serveurs pour exécuter les jobs
dans un pipeline. GitLab fournit des runners à utiliser, mais vous pouvez aussi utiliser vos propres serveurs comme runners.
- [jobs](https://docs.gitlab.com/ee/ci/jobs/) : tâche à exécuter dans un pipeline.
- [stage](https://docs.gitlab.com/ee/ci/yaml/README.html#stages) : un groupe de jobs connexes à exécuter dans un pipeline.

### Installation de votre runner 

Pour installer votre runner, veillez vous référer à la [documentation](https://docs.gitlab.com/runner/install/). Vous pouvez l'installer en local, ou via un conteneur (e.g. docker). Vous aurez besoin des informations présent sur votre projet dans gitlab : Settings / CI/CD / Specific Runners, Setup
a specific Runner manually.

- Pour l'URL copiez coller ce que vous trouverez à l’adresse précédente
- Pour le token copiez coller ce que vous trouverez à l’adresse précédente
- Pour le executor : _docker_ ou _shell_ en fonction

#### En local: 

	sudo curl --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64" 
	gitlab-runner register
	gitlab-runner install
	gitlab-runner start

#### En local, via docker: 

Installer un runner docker sur votre pc en suivant les instructions :
https://docs.gitlab.com/runner/install/docker.html

Lancer votre Runner à l’aide de la commande
sudo docker run -d --name gitlab-runner --restart always -v /srv/gitlab-runner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest

Si vous ne possédez pas docker, veuillez suivre les instructions d’installation de Docker pour votre système d’exploitation https://docs.docker.com/get-docker/. 
Installer ensuite le gitlab
Runner , puis faire la partie register runner
https://docs.gitlab.com/runner/register/index.html#docker . 

### Configuration de votre runner sur votre projet Gitlab

Une fois l'installation de votre runner terminée, vous devriez le voir apparaître dans votre projet sur Gitlab : Settings / CI/CD / Runners.

Retourner dans Settings / CI/CD / Specific Runners, Setup a specific Runner manually, et configurer votre Runner pour accepter les jobs non tagger : Cliquer sur le crayon pour éditer votre runner, puis cochez la case Run untagged jobs. Voila, votre runner est maintenant prêt!

Vous pouvez explorer le menu Settings / Integration et ajouter l’envoie d’un email automatique à votre adresse mail lorsque vous effectuer un push sur le repository, et lorsque votre pipeline de CI/CD obtient une erreur.

### Configuration de votre pipeline

GitLab utilise le fichier ".gitlab-ci.yml" pour faire fonctionner le pipeline de l'Intégration Continue pour chaque projet. Le fichier ".gitlab-ci.yml" doit se trouver dans le répertoire racine de votre projet. 

Créez un fichier ".gitlab-ci.yml", et essayez le contenue suivant. Vosu pouvez ensuite explorer votre pipeline (CI/CD / Pipelines). 

	job1:
	    stage: build 
	    script:
	    - echo "foo"
	job2:
	    stage: test 
	    script:
	    - echo "bar"
	job3:
	    stage: deploy 
	    script:
	    - echo "foobar"

Voici un autre exemple : 

       java:
  	stage: test
	script:
    	- mvn verify
  	artifacts:
    		when: always
    		reports:
      			junit:
        		- target/surefire-reports/TEST-*.xml
        		- target/failsafe-reports/TEST-*.xml
			
	test-jdk11:
	  stage: test
	  image: maven:3.6.3-jdk-11
	  script:
	    - 'mvn $MAVEN_CLI_OPTS clean org.jacoco:jacoco-maven-plugin:prepare-agent test jacoco:report'
	  artifacts:
	    paths:
	      - target/site/jacoco/jacoco.xml

	coverage-jdk11:
	  # Must be in a stage later than test-jdk11's stage.
	  # The `visualize` stage does not exist by default.
	  # Please define it first, or chose an existing stage like `deploy`.
	  stage: visualize
	  image: haynes/jacoco2cobertura:1.0.4
	  script:
	    # convert report from jacoco to cobertura
	    - 'python /opt/cover2cover.py target/site/jacoco/jacoco.xml src/main/java > target/site/cobertura.xml'
	    # read the <source></source> tag and prepend the path to every filename attribute
	    - 'python /opt/source2filename.py target/site/cobertura.xml'
	  needs: ["test-jdk11"]
	  dependencies:
	    - test-jdk11
	  artifacts:
	    reports:
	      cobertura: target/site/cobertura.xml

Aidez vous de la [documentation](https://docs.gitlab.com/ee/ci/yaml/), et définissez un pipeline similaire à celui que vous avez définit sur Jenkins dans la partie précédente.
