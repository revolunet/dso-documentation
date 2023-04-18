# Gestion des secrets sur DSO

La gestion des secrets est en cours de mise ne oeuvre. Le déploiement applicatif suit le principe gitOps et donc de "pousser" l'ensemble des déploiements et charts helm sur un repository GIT. Le problème est de pousser un secret kubernetes et qui se retrouve donc accessible.

Une première solution de mise en oeuvre avec [SOPS](https://github.com/mozilla/sops) est proposée. Ainsi, sur ce principe le secret est toujours poussé sur un repo git mais chiffré par une clé asymétrique.

Pour cela des paires de clés au format age ont été générées sur les différents clusters dont voici les clés publiques:
 * Cluster 4-7 : age1v34shlqv52vggpp54e3fn93rna2wek84s40lkv6wlzjun5xm
 * Cluster 4-8 : TO BE DONE
 * Cluster 4-5 : TO BE DONE

Afin de chiffrer un secret, il faut commencer par créer un secret de type SopsSecret par exemple :
```yaml
apiVersion: isindir.github.com/v1alpha3
kind: SopsSecret
metadata:
  name: mysecret-sops
spec:
  secretTemplates:
    - name: secret-exemple
      labels:
        label1: label1-value1
      annotations:
        key1: key1-value
      stringData:
        data-name0: data-value0
      data:
        data-name1: data-value1
    - name: token-exemple
      stringData:
        token: supersecrettoken
    - name: docker-login
      type: "kubernetes.io/dockerconfigjson"
      stringData:
        .dockerconfigjson: '{"auths":{"index.docker.io":{"username":"user","password":"pass","email":"toto@example.com","auth":"dXNlcjpwYXNz"}}}'
```

> Chaque élément du template donnera lieu à un secret dans kube une fois le secret décrypté 
Le secret secret-exemple sera crée avec les données suivantes
```yaml
data:
  data-name0: ZGF0YS12YWx1ZTA=
  data-name1: ZGF0YS12YWx1ZTE=
```

Ce fichier **ne doit pas** être commité et envoyé sur un repo git et rester en local.

Ensuite, il faut chiffrer ce fichier via SOPS avec la clé publique correspondant à l'environnement. Par exemple sur l'environnement 4 7 :

```bash
sops -e --age age1v34shlqv52vggpp54e3fn93rna2wek84s40lkv6wlzjun5xm6ekqemjhn3 --encrypted-suffix Templates secret.sops.yaml > secret.sops.enc.yaml
```

Le fichier chiffré doit conserver l'extension .yaml

Le contenu du fichier devient alors :

```yaml
apiVersion: isindir.github.com/v1alpha3
kind: SopsSecret
metadata:
    name: mysecret-sops
spec:
    secretTemplates:
        - name: ENC[AES256_GCM,data:GCTwnqz3qLWBltXkI1E=,iv:6s/89KUaymATUyyiavb1JQdndbvBY5XrBwdqg7Zp7nM=,tag:LcEyQU0/UvYWt7YCyGiFpw==,type:str]
          labels:
            label1: ENC[AES256_GCM,data:YKnixwbNsFPccmx7Kw==,iv:TQNNfHyvcJaXTnuNAi7iq/HHGpjtIN3SInxds1aWJpM=,tag:LCHNCjIQ8lrKRzlHIflYRA==,type:str]
          annotations:
            key1: ENC[AES256_GCM,data:IrcoTdCj0sS+tA==,iv:W5ccDu7jna7fP/ZfQ6cYaQX/uqU9PjKJ83PgJpHR9b0=,tag:4UDYI4WHXgipY8wXZu/NhA==,type:str]
          stringData:
            data-name0: ENC[AES256_GCM,data:t431CrKunuDACSw=,iv:pou2IIpBl6LeKloCC1yGzHA8Vkt/0Jo0nu8M4e+8XW0=,tag:kkuw1HXkSCS9f5K73MBEgw==,type:str]
          data:
            data-name1: ENC[AES256_GCM,data:p7ffMnDqVKj8Vog=,iv:KI07DuHBarC4du/sqrLus4o9s7o5knu/wu3W8ssO4e8=,tag:TgKXwVJJGEI9H5jWM5Ca4A==,type:str]
        - name: ENC[AES256_GCM,data:isdJvbWonL593lfI4w==,iv:bpHG0fsIXWcmJ3fCDebKXeFGWNrHfHRWTQ86e+Dgruw=,tag:HmgDZLLssR+roPBSsSrizw==,type:str]
          stringData:
            token: ENC[AES256_GCM,data:QYXc7S8EHkSblo7RDW9Ovw==,iv:9lJcVQ5EJR+LYVFX/0OUJ+uZqQx0kiL2Kze8OJ3fu0M=,tag:QDpVKlSS1jj+OnWzpfCW2Q==,type:str]
        - name: ENC[AES256_GCM,data:pHr2cwiFjGsIBZj+,iv:x1TkramaD0peRJe95n+r+ye5IWeeE630C0LwbVWJ154=,tag:7TiqbtURY7fn+9r2V7PlDA==,type:str]
          type: ENC[AES256_GCM,data:8ZBm++dOCx4xlmPW4bZagHMVua5gj7v7GVkEtBRX,iv:Y8HYgfO8Ae9SY3WYF/BYhKY9n6KESwQEHMNUPZfQd9o=,tag:LqlUy6FLcYAtYEa8qtx5NQ==,type:str]
          stringData:
            .dockerconfigjson: ENC[AES256_GCM,data:1TmIvfP95WReEpGqF2/ukxkvyFVdYbO3gda+oAtNZqwRZw749qvU8koYsi012s1/yhutll5v3ldqUYtr4sNuVS7TFVy2/qZ+ryiBaI8qUxt+kOx85eyfp36pJolwQtdQPNanRTLLkV4mf1JzSYOG6WAokkQ=,iv:X1jzTyp+CzTIowxH6gl2cIInk892cuO9/5JUkuCJdqI=,tag:DbtJe3pRo8TMrBO/gt4BDw==,type:str]
sops:
    kms: []
    gcp_kms: []
    azure_kv: []
    hc_vault: []
    age:
        - recipient: age1v34shlqv52vggpp54e3fn93rna2wek84s40lkv6wlzjun5xm6ekqemjhn3
          enc: |
            -----BEGIN AGE ENCRYPTED FILE-----
            YWdlLWVuY3J5cHRpb24ub3JnL3YxCi0+IFgyNTUxOSBTZFJGdkRzeVJFdDVUVjVS
            SmU0ekE0aGplVzhxZm5UTklzZnA3QlIwRzNNCjJha21CU0VEZkt3SHB0THlVZ1ZM
            VnlBSEZpbmJZVnlyY1VCTjdXZEtOR2cKLS0tIHBhRkprelpsQTJZWGphYUtRSlhJ
            ZnpFSDNYT0M3K294VmlBVitmN09nUlUKI+THCBdEkTnAElA3b0z4r8Nx1KcW7gks
            H5xJwqzzNn5C+UMy+v+Qn2hzg07juISBTDVcLtBDggVrZOAsh8kTMQ==
            -----END AGE ENCRYPTED FILE-----
    lastmodified: "2023-04-18T15:02:34Z"
    mac: ENC[AES256_GCM,data:Cpl1/GWn2LQivSo0qNTGyDxqCxm79DnomIpf1uDsoIuA5qqsluCUja0RLkEOm/fUD+UKzL8Muaqjo8+fbuKOvr4nfqaeARACPz377tdPEH55DHyg8Czv00OsxdHZ8C9BGeeSZr3YHDqQEKqQpK1zs7rBz/2adqD1SXrOFu+aiuQ=,iv:w+4DAXVAvD7IvDCBMTF+NfMRctp0dEWl+QsRJPsrd70=,tag:fWa0Sz3TlCQ2lIkVe6zE4Q==,type:str]
    pgp: []
    encrypted_suffix: Templates
    version: 3.7.3
```

Ce fichier peut être commité et envoyé sur un repo git car le contenu est chiffré.



