# Secret Kullanımı

Secret, parola, key, token gibi hassas bilgileri _cluster_ içerisinde saklamamıza yarayan bir Kubernetes objesidir. Örneğin bir uygulamamızın kullanacağı veri tabanı bağlantı bilgileri, bir API Key gibi bilgiler cluster'da bu şekilde saklanmalıdır.

Kullanımı ve görünüşü temelde Configmap'e çok benzese de farklılaştığı iki temel nokta vardır:

1. Configmap'in aksine, __namespace dışından erişilemezler.__
2. İçerisindeki veriyi base64 encoded olarak saklar, herhangi bir şeyi saklamaya imkan tanır (binary vs.)
3. Farklı secret tipleri vardır

Aşağıda `kubectl` ile bir secret oluşturacağız. `kubectl` ile secret oluşturmanın çok çeşitli yolları var. En çok kullanılanlar dosyadan oluşturmak ve doğrudan değerlerle oluşturmak.

İlk örnekte key-value şeklinde bir secret objesi oluşturalım:

```bash
kubectl create secret -n default generic my-secret --from-literal=password=abc123 --from-literal=username=user1
```
```bash
secret/my-secret created
```
Bu komutla kullanıcı adı "user1" olan ve parolası "abc123" olan, "my-secret" isminde bir Kubernetes _secret objesi_ oluşturuyoruz.

> ! `kubectl`'i dry-run olarak çalıştırmak için parametre sonunda `--dry-run=client` ekleyebilirsiniz.

Benzer şekilde, secret objelerini bir dosyadan da oluşturabiliriz.

```bash
echo "my-secret-password" > secret.txt
kubectl create secret -n default generic file-secret --from-file=secret.txt
```
İlk komut ile secret.txt isimli bir dosyaya "my-secret-password" yazdık ve ikinci komutla da bo dosyanın içeriğinden bir secret objesi oluşturduk.

Şimdi oluşturduğumuz secret objesini görüntüleyelim:
```bash
kubectl get secrets -n default
```
```bash
NAME          TYPE     DATA   AGE
file-secret   Opaque   1      3m25s
my-secret     Opaque   2      2m39s
```
Bir tanesi `file-secret` öbürü de ilk oluşturduğumuz `my-secret` objeleri. Fakat bu halde pek bir şey ifade etmiyor.
Bunların detaylarını `json` veya `yaml` olarak görüntüleyebiliriz.
```bash
kubectl get secret file-secret -o yaml
```
> `kubectl` kullanırken bir objenin çıktısını belirli bir formatta almak için `-o` parametresini kullanabilirsiniz. `-o yaml` veya `-o json` gibi.
```yaml
apiVersion: v1
data:
  secret.txt: bXktc2VjcmV0LXBhc3N3b3JkCg==
kind: Secret
metadata:
  creationTimestamp: "2025-02-09T20:41:36Z"
  name: file-secret
  namespace: default
  resourceVersion: "199568"
  uid: 47110152-2b9f-4408-a1bc-25c479624ed8
type: Opaque
```
Burada secret.txt içerisine yazdığımız "my-secret-password"ün base64 encoded halini görüyoruz. Bunu herhangi bir şekilde decode edebilirsiniz.

Dosya olarak uygulamak içinse:
```bash
kubectl apply -f secret.yaml
```
> Secret'ı YAML halde oluştururken değerleri base64 encode edip girmek istemiyorsanız `data` scopeunu `stringData` olarak değiştirebilrisiniz. (örnek: `stringData.password: abc123`)

Eğer bilgisayarınızda `yq` veya `jq` yüklüyse şu şekilde parse edebilirsiniz:
```bash
kubectl get secret file-secret -o yaml | yq '.data."secret.txt"' | base64 -d

## veya
kubectl get secret file-secret -o json | jq -r '.data["secret.txt"]' | base64 -d
```
Bu komut size doğrudan decoded base64 değeri verecek, yazılan metni okumanızı sağlayacaktir.
> Ne olduğunu bilmediğiniz base64 kodları terminalinizde çalıştırmrayın.

## Secret Tipleri
Secretları kullanırken, "type" alanında Secret'ın tipini belirtebiliriz:

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: my-secret
  namespace: default
data:
  password: YWJjMTIz
  username: dXNlcjE=
```

| Tip                                   | Kullanım                                   |
| ------------------------------------- |---------------------------------------- |
| `Opaque`                              | Kullanıcı tanımlı veri            |
| `kubernetes.io/service-account-token` | ServiceAccount token                    |
| `kubernetes.io/dockercfg`             | serialized `~/.dockercfg` dosyası          |
| `kubernetes.io/dockerconfigjson`      | serialized `~/.docker/config.json` dosyası |
| `kubernetes.io/basic-auth`            | basic-auth kimlik doğrulaması için    |
| `kubernetes.io/ssh-auth`              | SSH kimlik doğrulaması için   |
| `kubernetes.io/tls`                   | Server veya client TLS sertifikası         |
| `bootstrap.kubernetes.io/token`       | bootstrap token data                    |

Secret tipleri, nerede nasıl kullanmak istediğinize göre değişkenlik gösterebilir. Her birisi için usecase ve kullanımlarını anlatan bir sayfayı da aşağıda bulabilirsiniz.
#### [Secret Tipleri Örnekleri](secret_ornek.yaml)

## Deployment'ta secret kullanımı
Secretları oluşturduktan kullanabilmek için bunları Podlara bağlamamız gerekiyor.
Pod'un secret'a erişlebilmnesi için ya bir _environment variable_ olarak eklememiz ya da bir dosya olarak bağlamamız gerekiyor.

Tabii içerideki uygulamanın secretları kullanım mekanizması da önemli. Bazı uygulamalar environment variable olarak tercih ederken bazıları dosya olarak tercih ediyor. İki örneği de yapalım:
```yaml

```