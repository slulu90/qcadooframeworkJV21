# Migration Notes — Qcadoo Framework JV21

## Contexte

Migration de **Qcadoo Framework 1.5-SNAPSHOT** de Java 8 vers Java 21.
Fork : https://github.com/slulu90/qcadooframeworkJV21
Branche : migration/java-21

Environnement : Windows 10 Pro 22H2, JDK Temurin 21.0.11, Maven 3.9.16

---

## Statut de compilation

- **BUILD SUCCESS** : 23/23 modules sous Java 21
- Tests unitaires : Mockito 1.x / PowerMock 1.5.x incompatibles Java 21 (problème préexistant)

---

## Modifications pom.xml racine

### AspectJ
- Plugin : `org.codehaus.mojo:aspectj-maven-plugin:1.7` → `dev.aspectj:aspectj-maven-plugin:1.14.1`
- aspectjtools / aspectjrt / aspectjweaver : `1.8.13` → `1.9.21.2`
- complianceLevel / source / target : `1.8` → `21`
- Ajout : `<Xlint>ignore</Xlint>`

### Dépendances ajoutées
- `javax.annotation:javax.annotation-api:1.3.2` (PostConstruct supprimé du JDK en Java 11)
- `com.sun.activation:jakarta.activation:1.2.2` (javax.activation supprimé du JDK en Java 11)

### dependencyManagement — exclusions JPMS (split packages)
- `xml-apis:xml-apis:1.4.01:provided`
- `xml-apis:xml-apis:1.3.04:provided`
- `xml-apis:xml-apis-ext:1.3.04:provided`
- `stax:stax-api:1.0.1:provided`

### Exclusions sur esapi (split packages AspectJ)
Exclusions ajoutées sur `org.owasp.esapi:esapi` :
- `xerces:xercesImpl`
- `xom:xom`
- `org.apache.xmlgraphics:batik-css`
- `net.sourceforge.htmlunit:neko-htmlunit`
- `xml-apis:xml-apis`
- `xml-apis:xml-apis-ext`

---

## Corrections Java source

### DefaultPlugin.java (qcadoo-plugin)
Import explicite ajouté pour lever l'ambiguïté avec `java.lang.Module` (Java 9+) :
`import com.qcadoo.plugin.api.Module;`

### Commons Lang ObjectUtils → java.util.Objects
Fichiers modifiés (9 occurrences) :
- `qcadoo-commons/...functional/Either.java`
- `qcadoo-report/...pdf/layout/VerticalLayout.java`
- `qcadoo-security/...hooks/UserModelHooks.java`

Remplacements :
- `ObjectUtils.equals()` → `Objects.equals()`
- `ObjectUtils.hashCode()` → `Objects.hashCode()`
- `ObjectUtils.toString()` → `Objects.toString(x, "")`

### DataDefinitionImpl.java (qcadoo-model)
`entityClass.newInstance()` → `entityClass.getDeclaredConstructor().newInstance()`
Ajout de `NoSuchMethodException` et `InvocationTargetException` dans le catch.

### ButtonComponentPattern.java / FilterValueHolderImpl.java (qcadoo-view)
`new Integer(String)` → `Integer.parseInt(String)`

### DefaultLocaleResolverImpl.java (qcadoo-tenant) / ReportDevelopmentController.java (qcadoo-report)
`new Locale(String)` → `Locale.forLanguageTag(String)`

### DefaultPluginDescriptorParser.java (qcadoo-plugin)
`new URL(String)` → `java.net.URI.create(String).toURL()`

---

## Dépréciations intentionnellement conservées

### APIs Qcadoo internes (SearchCriteriaBuilder)
Méthodes dépréciées de l'ancienne API Hibernate Criteria encapsulée par Qcadoo.
Ces méthodes sont utilisées par l'implémentation interne (SearchCriteriaImpl, GridComponentState, etc.)
et ne peuvent pas être remplacées sans refactoring majeur de l'API Qcadoo elle-même.
- `createAlias()`, `isEq()`, `isLe()`, `isLt()`, etc. dans SearchCriteriaImpl.java
- `createAlias()` dans DictionaryServiceImpl.java, EntityListImpl.java, GridComponentState.java
- `isEnabledOrEnabling()` dans EntityHookDefinitionImpl.java
- `performEvent()` dans CrudServiceImpl.java

### APIs Spring Security internes dépréciées
Nécessiterait une mise à jour de Spring Security 3.x vers 5.x+ :
- `successfulAuthentication()`, `getAuthentication()`, `GrantedAuthorityImpl` dans qcadoo-security

### Runtime.exec(String) (DefaultPluginServerManager.java)
Déprécié mais fonctionnel — remplacement par `Runtime.exec(String[])` possible ultérieurement.

---

## Warning Maven non bloquant

`dependencyManagement.dependencies.dependency.(groupId:artifactId:type:classifier) must be unique: xml-apis:xml-apis:jar`
→ Doublon `xml-apis:1.4.01` vs `xml-apis:1.3.04` dans le dependencyManagement.
Non bloquant pour la compilation — à corriger dans une prochaine itération.

---

## Git log (branche migration/java-21)

`fix: replace deprecated Locale(String), URL(String), BigDecimal.setScale with modern equivalents`
`fix: replace deprecated Integer(String) constructor with Integer.parseInt()`
`fix: replace deprecated Class.newInstance() with getDeclaredConstructor().newInstance()`
`fix: replace deprecated Commons Lang ObjectUtils methods with java.util.Objects`
`chore: compile Java 21 - fix aspectj, xml-apis, javax.annotation, javax.activation, Module ambiguity + update .gitignore`