## Jenkins

- [Env Var](#env-var)
- [Workspace](#workspace)
- [Reference func in file](#reference-func-in-file)
- [Best practices](#best-practices)
- Read more
  - [Jenkins CheatSheet — Know The Top Best Practices of Jenkins](https://medium.com/edureka/jenkins-cheat-sheet-e0f7e25558a3)

### Env Var

- `withEnv(["env=value]) {}` block can overwrite any env vars.
- Vars set using `environment {}` block **CANNOT** be assigned with another value using imperative `env.VAR = "value"` assignment.
- Imperative `env.VAR = "value"` assignment can overwrite only env vars created using imperative assignment.

```groovy
script {
  withEnv(["FOO=foobar"]) { // it can overwrite any env variable
      echo "FOO = ${env.FOO}" // prints "FOO = foobar"
  }
}
```

### Workspace

The workspace directory is where Jenkins builds your project: it contains the source code Jenkins checks out, plus any files generated by the build itself. This workspace is reused for each successive build—there is only ever one workspace directory per project, and the disk space it requires tends to be relatively stable.

### Reference func in file

Example.Groovy
```groovy
def exampleMethod() {
    //do something
}

def otherExampleMethod() {
    //do something else
}
return this
```

JenkinsFile

```groovy
node {
    def rootDir = pwd()
    def example = load "${rootDir}@script/Example.Groovy "
    example.exampleMethod()
    example.otherExampleMethod()
}
```

### Best practices

#### Combine multiple steps into one

```groovy
// do this
sh """
  echo Installing foundational dependency...
  ./batect install-foundational-dependency
  echo Bootstraping
  ./batect bootstrap
"""

// not this as it will require connections and resources on the agent and master to be created and cleaned up
// echo "Installing foundational dependency..."
// sh "./batect install-foundational-dependency"
// echo "Bootstraping"
// sh "./batect bootstrap"
```

