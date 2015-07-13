
ixa-pipe-pos
============

ixa-pipe-pos is a multilingual Part of Speech tagger, currently offering pre-trained models for English, Galician and Spanish. ixa-pipe-pos is part of IXA pipes, a multilingual set of NLP tools developed
by the IXA NLP Group [http://ixa2.si.ehu.es/ixa-pipes]. **Current version is 1.4.1**.

Please go to [http://ixa2.si.ehu.es/ixa-pipes] for general information about the IXA
pipes tools but also for **official releases, including source code and binary
packages for all the tools in the IXA pipes toolkit**.

This document is intended to be the **usage guide of ixa-pipe-pos**. If you really need to clone
and install this repository instead of using the releases provided in
[http://ixa2.si.ehu.es/ixa-pipes], please scroll down to the end of the document for
the [installation instructions](#installation).

## TABLE OF CONTENTS

1. [Overview of ixa-pipe-pos](#overview)
  + [Distributed resources](#resources)
  + [Distributed models](#models)
2. [Usage of ixa-pipe-pos](#usage)
  + [POS tagging/lemmatizing](#tagging)
  + [Training your own models](#training)
  + [Evaluation](#evaluation)
3. [API via Maven Dependency](#api)
4. [Git installation](#installation)
5. [Adding your language](#extend)

## OVERVIEW

ixa-pipe-pos provides POS tagging and lemmatization several languages. We
provide Perceptron (Collins 2002) and Maximum Entropy (Ratnapharki 1999) POS tagging models:

+ **POS tagging models for English** trained and evaluated using the WSJ treebank with the usual
partitions (Ratnapharki 1999).
+ **POS tagging models for Spanish** trained and evaluated using the [Ancora corpus](http://clic.ub.edu/corpus/ancora) via 10-fold cross-validation.
+ **POS tagging models for Galician** trained and evaluated using the [CTAG corpus](http://sli.uvigo.es/CTAG/) via 10-fold cross-validation.
+ **Dictionary-based lemmatization** for English, Galician and Spanish.
+ **Multiword detection** for Spanish and Galician.
+ **Post-processing** of probabilistic model pos tags using monosemic dictionaries (Spanish and Galician).

To avoid duplication of efforts, we use and contribute to the machine learning API provided by the [Apache OpenNLP project](http://opennlp.apache.org). Additionally, we have added other features such as dictionary-based lemmatization, multiword and clitic pronoun treatment, post-processing via tag dictionaries, etc., as described below.

**ixa-pipe-pos is distributed under Apache License version 2.0 (see LICENSE.txt for details)**.

### Resources

**The contents of this package are required for compilation**. Therefore, please get and **unpack** the contents of this tarball in the **src/main/resources/** directory inside ixa-pipe-pos before compilation.

The following resources **include lemmatization and multiword dictionaries**, and are available in the [pos-resources.tgz](http://ixa2.si.ehu.es/ixa-pipes/models/pos-resources.tgz)
package. Note that the dictionaries come with their own licences, please do comply with them:

+ **Lemmatizer Dictionaries**: "word\tablemma\tabpostag" dictionaries binarized as Finite State Automata using the  [morfologik-stemming project](https://github.com/morfologik/morfologik-stemming):
  + english.dict, galician.dict, spanish.dict. Via API you can also pass a plain text dictionary of the same tabulated format.

+ **Multiword Dictionaries**: "multi#word\tab\multi#lemma\tab\postag\tabambiguity" dictionaries to detect multiword expressions. Currently vailable:
  + es-locutions.dict for **Spanish** and gl-locutions.dict in **Galician**.

+ **Monosemic Tag Dictionaries**: the monosemic versions of the lemmatizer dictionaries. This is used for post-processing the results of the POS tagger if and when the option **--dictag** is activated in CLI. Currently available:
  + spanish-monosemic.dict, galician-monosemic.dict.

**It is required before compilation** to download the package, copy it and untar it into the src/main/resources directory.

### Models

+ Latest models: [pos-models-1.4.x](http://ixa2.si.ehu.es/ixa-pipes/models/pos-models-1.4.0.tar.gz).
+ 1.3.x models: [pos-models-1.3.x](http://ixa2.si.ehu.es/ixa-pipes/models/pos-models-1.3.0.tgz).

+ **English POS Models**:
  + Penn Treebank: **en-maxent-100-c5-baseline-penn.bin**: 96.81

+ **Spanish POS Models**: We provide two Perceptron models:
  + Ancora with automatic dictionary created from training data **es-perceptron-baseline-autodict01-ancora.bin**: 97.56 word accuracy in 10-fold cross validation.
  + Ancora: **es-perceptron-baseline-ancora.bin**: With Baseline features, this is slightly faster and obtains 97.35 word accuracy in 10-fold cross validation.

+ **Galician POS Model**:
  + CTAG corpus with an automatic dictionary created from training data **gl-perceptron-baseline-autodict05-ctag.bin**: 98.55 word accuracy in 10-fold cross validation.

Remember that for Galician and Spanish the output of the statistical models can be post-processed using the monosemic dictionaries provided via the **--dictag** CLI option.

## USAGE

ixa-pipe-pos provides 4 basic functionalities:

1. **tag**: reads a NAF document containing *wf* elements and creates *term* elements with the morphological information.
2. **train**: trains new models for with several options
   available (read trainParams.properties file for details).
3. **eval**: evaluates a trained model with a given test set.
4. **cross**: perform cross-validation evaluation.

Each of these functionalities are accessible by adding (tag|train|eval|cross) as a
subcommand to ixa-pipe-pos-$version.jar. Please read below and check the -help
parameter:

````shell
java -jar target/ixa-pipe-pos-$version.jar (tag|train|eval|cross) -help
````

### Tagging

If you are in hurry, just execute:

````shell
cat file.txt | ixa-pipe-tok | java -jar $PATH/target/ixa-pipe-pos-$version.jar tag -m model.bin
````

If you want to know more, please follow reading.

ixa-pipe-pos reads NAF documents containing *wf* elements via standard input and outputs NAF
through standard output. The NAF format specification is here:

(http://wordpress.let.vupr.nl/naf/)

You can get the necessary input for ixa-pipe-pos by piping it with
[ixa-pipe-tok](https://github.com/ixa-ehu/ixa-pipe-tok).

There are several options to tag with ixa-pipe-pos:

+ **model**: it is **required** to provide the model to do the tagging.
+ **lang**: choose between en and es. If no language is chosen, the one specified
  in the NAF header will be used.
+ **beamSize**: choose beam size for decoding. There is no definitive evidence
  that using larger or smaller beamsize actually improves accuracy. It is known
  to slow things down considerably if beamsize is set to 100, for example.
+ **multiwords**: activates the multiword detection option.
+ **dictag**: post-process the Statistical POS tagger output via a monosemic
  postag dictionary.

**Tagging Example**:

````shell
cat file.txt | ixa-pipe-tok | java -jar $PATH/target/ixa-pipe-pos-$version.jar tag -m model.bin
````

### Training

To train a new model, you just need to pass a training parameters file as an
argument. Every training option is documented in the template trainParams.properties file.

**Example**:

````shell
java -jar target/ixa.pipe.pos-$version.jar train -p trainParams.properties
````

### Evaluation

To evaluate a trained model, the eval subcommand provides the following
options:

+ **model**: input the name of the model to evaluate.
+ **testSet**: testset to evaluate the model.
+ **evalReport**: choose the detail in displaying the results:
  + **brief**: it just prints the word accuracy.
  + **detailed**: detailed report with confusion matrixes and so on.
  + **error**: print to stderr all the false positives.
+ **beamsize**: choose beamsize for decoding.

**Example**:

````shell
java -jar target/ixa.pipe.pos-$version.jar eval -m test-pos.bin -l en -t test.data
````

## API

The easiest way to use ixa-pipe-pos programatically is via Apache Maven. Add
this dependency to your pom.xml:

````shell
<dependency>
    <groupId>eus.ixa</groupId>
    <artifactId>ixa-pipe-pos</artifactId>
    <version>1.4.0</version>
</dependency>
````

## JAVADOC

The javadoc of the module is located here:

````shell
ixa-pipe-pos/target/ixa-pipe-pos-$version-javadoc.jar
````
## Module contents

The contents of the module are the following:

    + formatter.xml           Apache OpenNLP code formatter for Eclipse SDK
    + pom.xml                 maven pom file which deals with everything related to compilation and execution of the module
    + src/                    java source code of the module and required resources
    + trainParams.properties      A template properties file containing documention
    + Furthermore, the installation process, as described in the README.md, will generate another directory:
    target/                 it contains binary executable and other directories


## INSTALLATION

Installing the ixa-pipe-pos requires the following steps:

If you already have installed in your machine the Java 1.7+ and MAVEN 3, please go to step 3
directly. Otherwise, follow these steps:

### 1. Install JDK 1.7 or JDK 1.8

If you do not install JDK 1.7+ in a default location, you will probably need to configure the PATH in .bashrc or .bash_profile:

````shell
export JAVA_HOME=/yourpath/local/java8
export PATH=${JAVA_HOME}/bin:${PATH}
````

If you use tcsh you will need to specify it in your .login as follows:

````shell
setenv JAVA_HOME /usr/java/java8
setenv PATH ${JAVA_HOME}/bin:${PATH}
````

If you re-login into your shell and run the command

````shell
java -version
````

You should now see that your JDK is 1.7+

### 2. Install MAVEN 3

Download MAVEN 3 from

````shell
wget http://apache.rediris.es/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
````

Now you need to configure the PATH. For Bash Shell:

````shell
export MAVEN_HOME=/home/ragerri/local/apache-maven-3.0.5
export PATH=${MAVEN_HOME}/bin:${PATH}
````

For tcsh shell:

````shell
setenv MAVEN3_HOME ~/local/apache-maven-3.0.5
setenv PATH ${MAVEN3}/bin:{PATH}
````

If you re-login into your shell and run the command

````shell
mvn -version
````

You should see reference to the MAVEN version you have just installed plus the JDK 7 that is using.

### 3. Get module source code

If you must get the module source code from here do this:

````shell
git clone https://github.com/ixa-ehu/ixa-pipe-pos
````

### 4. Download the Resources

You will need to download the resources and copy them to ixa-pipe-pos/src/main/resources/
for the module to work properly:

Download the resources and untar the archive into the src/main/resources directory:

````shell
cd ixa-pipe-pos/src/main/resources
wget http://ixa2.si.ehu.es/ixa-pipes/models/pos-resources.tgz
tar xvzf pos-resources.tgz
````
The pos-resources contains the required dictionaries for ixa-pipe-pos to run.

### 5. Compile

````shell
cd ixa-pipe-pos
mvn clean package
````

This step will create a directory called target/ which contains various directories and files.
Most importantly, there you will find the module executable:

ixa-pipe-pos-$version.jar

This executable contains every dependency the module needs, so it is completely portable as long
as you have a JVM 1.7 or newer installed.

To install the module in the local maven repository, usually located in ~/.m2/, execute:

````shell
mvn clean install
````

## Extend

To add your language to ixa-pipe-pos the following steps are required:

+ Create lemmatizer and (if required) multiword and monosemic dictionaries following the format of those distributed in **pos-resources.tgz**.
  + **Create binary dictionaries (FSA):** Starting from the plain text tabulated dictionaries, do the following steps:
    + Get Morfologik standalone binary: http://sourceforge.net/projects/morfologik/files/morfologik-stemming/
    + java -jar morfologik-tools-1.6.0-standalone.jar tab2morph --annotation "*" -i
    ~/javacode/ixa-pipe-pos/pos-resources/lemmatizer-dicts/freeling/es-lemmatizer.dict -o spanish.morph
    + java -jar morfologik-tools-1.6.0-standalone.jar fsa_build -i spanish.morph -o spanish.dict
    + **Create a *.info file like spanish.info**
+ **Modify the classes** CLI, Resources and Annotate; if multiword is required also MultiWordMatcher; if monosemic dictionaries for post-processing also MorfologikMorphoTagger) adding for your language the same information that it is available for other languages.
+ Train a model. **It is crucial that the tagset of the dictionaries and corpus be the same**. Also it is recommended to train a model with an external dictionary (the external tag dictionary needs to be in opennlp tag format).
+ Add documentation to this README.md.
+ **Do a pull request** to merge the changes with your new language.
+ Send us the resources and models created if you want them to be distributed with ixa-pipe-pos (Apache License 2.0 is favoured).

## Contact information

````shell
Rodrigo Agerri
IXA NLP Group
University of the Basque Country (UPV/EHU)
E-20018 Donostia-San Sebastián
rodrigo.agerri@ehu.eus
````
