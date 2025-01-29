# kong-konnect-apiops

## Setup konnect
* Create Control plane named "training"
* Create dataplane
* create .deck.yaml from template

## Connect deck to Konnect Cloud
```
deck --config .deck.yaml gateway ping
```

## Manual deck 

### oas to deck 
```
deck file openapi2kong -s bu1/httpbin/oas.yaml >bu1/httpbin/.generated/1_base_kong.yaml
diff /dev/null bu1/httpbin/.generated/1_base_kong.yaml 
```

### deck add-plugins

```
deck file add-plugins -s bu1/httpbin/.generated/1_base_kong.yaml -o bu1/httpbin/.generated/2_with_plugins_kong.yaml bu1/httpbin/kong/plugins.yaml
diff bu1/httpbin/.generated/1_base_kong.yaml bu1/httpbin/.generated/2_with_plugins_kong.yaml
```


### deck file patch (for environment)
```
deck file patch -s bu1/httpbin/.generated/2_with_plugins_kong.yaml -o bu1/httpbin/.generated/3_with_env_patch_kong.yaml bu1/httpbin/environments/helm-tpl/patches.yaml
diff bu1/httpbin/.generated/2_with_plugins_kong.yaml bu1/httpbin/.generated/3_with_env_patch_kong.yaml
```

### deck add-tags


```
deck file add-tags -s bu1/httpbin/.generated/3_with_env_patch_kong.yaml -o bu1/httpbin/.generated/4_with_tags.yaml managed-by-deck
diff bu1/httpbin/.generated/3_with_env_patch_kong.yaml bu1/httpbin/.generated/4_with_tags.yaml
```

### deck file namespace to add route prefix

```
deck file namespace -s bu1/httpbin/.generated/4_with_tags.yaml -o bu1/httpbin/.generated/5_with_namespace.yaml --path-prefix=/httpbin
diff bu1/httpbin/.generated/4_with_tags.yaml bu1/httpbin/.generated/5_with_namespace.yaml

```

### deck file validate

```
deck file validate  bu1/httpbin/.generated/5_with_namespace.yaml
```

## deck with KIC

### deck file kong2kic

deck file kong2kic -s bu1/httpbin/.generated/5_with_namespace.yaml -o bu1/httpbin/helm/httpbin-chart/templates/kong-kic.yaml

### helm template

set in values.yaml of helm chart `kong.customHeader` (defaulting to set-by-helm)

```
helm template bu1/httpbin/helm/httpbin-chart  | grep -n10 set-by-helm
```

See the value is replaced by helm. This may only work for strings. Would require more research as of now.

## deck with konnect/CP

exchange env path in step add-plugins with dev instead of helm-tpl

### deck gateway sync

```
deck --config .deck.yaml gateway sync  bu1/httpbin/.generated/5_with_namespace.yaml
```

### deck gateway reset 

```
deck --config .deck.yaml gateway
```

## All together

```
deck file openapi2kong -s bu1/httpbin/oas.yaml >bu1/httpbin/.generated/1_base_kong.yaml
deck file add-plugins -s bu1/httpbin/.generated/1_base_kong.yaml -o bu1/httpbin/.generated/2_with_plugins_kong.yaml bu1/httpbin/kong/plugins.yaml
deck file patch -s bu1/httpbin/.generated/2_with_plugins_kong.yaml -o bu1/httpbin/.generated/3_with_env_patch_kong.yaml bu1/httpbin/environments/helm-tpl/patches.yaml
deck file add-tags -s bu1/httpbin/.generated/3_with_env_patch_kong.yaml -o bu1/httpbin/.generated/4_with_tags.yaml managed-by-deck
deck file namespace -s bu1/httpbin/.generated/4_with_tags.yaml -o bu1/httpbin/.generated/5_with_namespace.yaml --path-prefix=/httpbin
deck file validate  bu1/httpbin/.generated/5_with_namespace.yaml
```
## Test httpbin

`curl localhost:8000/httpbin/headers -H "myheader: test"`
`curl localhost:8000/httpbin/delay/5 -H "myheader: test"`