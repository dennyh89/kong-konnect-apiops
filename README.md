# kong-konnect-apiops

## Setup konnect
* Create Control plane named "training"
* Create dataplane

## Connect deck to Konnect Cloud
* use flags
```
deck gateway ping \
  --konnect-control-plane-name="training" \
  --konnect-addr="https://eu.api.konghq.com" \
  --konnect-token="${KPAT}"
```
* or config file:
  * copy and modify .deck.yaml.template to .deck.yaml
  * `deck --config .deck.yaml gateway ping`

## Manual deck 

### oas to deck 
```
deck file openapi2kong -s bu1/httpbin/oas.yaml >bu1/httpbin/.generated/1_base_kong.yaml
```

### deck add-plugins

```
deck file add-plugins -s bu1/httpbin/.generated/1_base_kong.yaml -o bu1/httpbin/.generated/2_with_plugins_kong.yaml bu1/httpbin/kong/plugins.yaml
diff bu1/httpbin/.generated/1_base_kong.yaml bu1/httpbin/.generated/2_with_plugins_kong.yaml
```


### deck file patch (for environment)
```
deck file patch -s bu1/httpbin/.generated/2_with_plugins_kong.yaml -o bu1/httpbin/.generated/3_with_env_patch_kong.yaml bu1/httpbin/environments/dev/patches.yaml
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

### deck gateway sync

```
deck --config .deck.yaml gateway sync  bu1/httpbin/.generated/5_with_namespace.yaml
```

## All together

```
deck file openapi2kong -s bu1/httpbin/oas.yaml >bu1/httpbin/.generated/1_base_kong.yaml
deck file add-plugins -s bu1/httpbin/.generated/1_base_kong.yaml -o bu1/httpbin/.generated/2_with_plugins_kong.yaml bu1/httpbin/kong/plugins.yaml
deck file patch -s bu1/httpbin/.generated/2_with_plugins_kong.yaml -o bu1/httpbin/.generated/3_with_env_patch_kong.yaml bu1/httpbin/environments/dev/patches.yaml
deck file add-tags -s bu1/httpbin/.generated/3_with_env_patch_kong.yaml -o bu1/httpbin/.generated/4_with_tags.yaml managed-by-deck
deck file namespace -s bu1/httpbin/.generated/4_with_tags.yaml -o bu1/httpbin/.generated/5_with_namespace.yaml --path-prefix=/httpbin
deck file validate  bu1/httpbin/.generated/5_with_namespace.yaml
```
## Test httpbin

`curl localhost:8000/httpbin/headers -H "myheader: test"`
`curl localhost:8000/httpbin/delay/5 -H "myheader: test"`
