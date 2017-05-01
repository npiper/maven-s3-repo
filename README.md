# Maven Repo is S3


## Create storage

```
aws s3 mb s3://solveapuzzle-repo
```

## Create groups, assign users

Create a new group with permission to access the Repo, assign an existing user to it.
```
aws iam create-group --group-name solveapuzzle-repo
aws iam attach-group-policy --group-name solveapuzzle-repo --cli-input-json ./s3_policy.json
aws iam add-user-to-group --group-name solveapuzzle-repo --user-name neil

```


## Modify the local settings.xml with logon info

Add the AWS access key for the new user to your local settings.xml.
Group administration supports allowing multiple users access, read only also possible vs. writable. A potential way to organise is to allow read only access to developers, and writable access to build users to enforce that snapshots or releases are always through a CI/CD toolchain.

```
<settings>
  <servers>
    <server>
      <id>solveapuzzle-repo</id>
      <username>AKIAI9NNNJD5D7X4TUVA</username>
      <password>t5tZQCwuaRhmlOXfbGE5aTBMFw34iFyxfCEr32av</password>
    </server>
    [...]
  </servers>
  [...]
</settings>
```

### Configure a parent pom.xml to use the Repo

Ideally put this in a parent POM so repository is used consistently. The following fragment supports a snapshot and release repo for deployment and a repo for pulling down dependencies.

```
<project>
  <distributionManagement>
    <snapshotRepository>
      <id>solveapuzzle-repo</id>
      <url>s3://solveapuzzle-repo/snapshot</url>
    </snapshotRepository>
    <repository>
      <id>solveapuzzle-repo</id>
      <url>s3://solveapuzzle-repo/release</url>
    </repository>
  </distributionManagement>
  <repositories>
    <repository>
      <id>solveapuzzle-repo</id>
      <url>s3://solveapuzzle-repo/release</url>
    </repository>
  </repositories>
  [...]
</project>

```

### Dependency in parent pom to use S3 Wagon to connect

S3 Wagon library can be used for communication to the Repository. Add this to the parent POM as a dependency.

```
<project>
  <build>
    <extensions>
      <extension>
        <groupId>org.kuali.maven.wagons</groupId>
        <artifactId>maven-s3-wagon</artifactId>
        <version>1.2.1</version>
      </extension>
    </extensions>
    [...]
  </build>
</project>
```

## References

http://www.yegor256.com/2015/09/07/maven-repository-amazon-s3.html

https://hub.github.com/

Google AppEngine Repo option (Max ~32Mb on free version)
https://github.com/renaudcerrato/appengine-maven-repository
