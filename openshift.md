# Starter commands

## Auto detect

```
oc new-app path/to/source/code
```

**openshift searches for `origin` and try to connect if blocked it creates as a binary build**

## Docker strategy

```
oc new-app /path/to/source/code --strategy=docker
```

## From template

```
oc new-app -f examples/sample-app/application-template-stibuild.json
```

