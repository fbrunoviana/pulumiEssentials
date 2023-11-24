## Pulumi & AWS - Como esse arquivo foi escrito
## Guelo
### Instalacao do Pulumi

MacOS: 
```
brew install pulumi/tap/pulumi
```

Linux:
```
curl -fsSL https://get.pulumi.com | sh
```

### Configurando Pulumi para acessar a sua conta AWS

```bash
export AWS_ACCESS_KEY_ID=<YOUR_ACCESS_KEY_ID> 
export AWS_SECRET_ACCESS_KEY=<YOUR_SECRET_ACCESS_KEY>
```

### Setup 

( Esse passo não é necessario se vc clonou o esse repo)

```
mkdir quickstart && cd quickstart && pulumi new aws-python
```

```
project name: (quickstart)
project description: (A minimal AWS Pulumi program)
Created project 'quickstart'
```

Ao ser perguntado sobre *stack name* clique em `ENTER`

Em `aws:region:` aperte  `ENTER` 

### Arquivo '\_\_main__.py'

Todas as alteracões são feitas no arquivo: \_\_main__.py 

Por padrão ele vem escrito da seguinte forma:

```python
import pulumi
from pulumi_aws import s3

# Create an AWS resource (S3 Bucket)
bucket = s3.Bucket('my-bucket')

# Export the name of the bucket
pulumi.export('bucket_name', bucket.id)
```

### Objetivo #1 criando uma maquina ec2

Minha primeira mudança foi alterar o import adicionando o ec2 

```python
from pulumi_aws import s3, ec2
```
Logo depois criei uma variavel que recebia o ec2.Instance 

```python
ec2_instance = ec2.Instance("web-app",
                            ami="ami-053b0d53c279acc90",
                            instance_type='t3.nano',
                            tags={
                                'Name': 'web-app'
                            })
```

Onde: 
- `web-app` é o nome da aplicacão;
- `ami` é o ami-id da imagem;
- `instance_type` é o tipo da instancia;
- `tags` é um dicionario que adiciona o valor Name como web-app

Para exibirmos o ip publico da vm podemos adicionar:

```python
output_public_ip.append(ec2_instance.public_ip)
```

Logo o condigo completo ficou da seguinte forma:

```python
from pulumi_aws import s3, ec2
ec2_instance = ec2.Instance("web-app",
                            ami="ami-053b0d53c279acc90",
                            instance_type='t3.nano',
                            tags={
                                'Name': 'web-app'
                            })
output_public_ip.append(ec2_instance.public_ip)
```
Voce pode testar com o comando:
```shell
pulumi up
```

Após os testes voce pode deletar a VM com o comando:
```shell
pulumi destroy 
```

### Objetivo #2 criando um secrect group

Agora vamos criar um secrect group como duas regras:

Criando o secrect group:

```python
sg = ec2.SecurityGroup('web-svc-gro', description="Security group for web service")
```

Criando regra para acesso ssh
```python
allow_ssh = ec2.SecurityGroupRule("AllowSSH", type="ingress", 
                                  from_port=22, to_port=22, 
                                  protocol="tcp", 
                                  cidr_blocks=["0.0.0.0/0"], 
                                  security_group_id=sg.id)
```

Criando regra para acesso http
```python
allow_http = ec2.SecurityGroupRule("AllowHTTP", type="ingress",
                                   from_port=80, to_port=80,
                                   protocol="tcp",
                                   cidr_blocks=["0.0.0.0/0"], 
                                   security_group_id=sg.id)
```

Criando regra para acesso all egress
```python
allow_all = ec2.SecurityGroupRule("AllowALL", type="egress",
                                  from_port=0, to_port=0,
                                  protocol="-1",
                                  cidr_blocks=["0.0.0.0/0"],
                                  security_group_id=sg.id)
```
Configurando a instancia:

```python
ec2_instance = ec2.Instance("web-app",
                            ami="ami-053b0d53c279acc90",
                            instance_type='t3.nano',
                            vpc_security_group_ids=[sg.id], # Mude aqui
                            tags={
                                'Name': 'web-app'
                            })
```
Voce pode testar com o comando:
```shell
pulumi up
```

Após os testes voce pode deletar a VM com o comando:
```shell
pulumi destroy 
```
### Objetivo #3 Alterando o output para exibir o DNS.

Vamos adicionar o output para exibir o DNS.

```python
pulumi.export('instance_url', ec2_instance.public_dns)
```

Voce pode testar com o comando:
```shell
pulumi up
```

Em Outputs:
A linha será exibida:

instance_url: "ec2-44-203-52-241.compute-1.amazonaws.com"

Após os testes voce pode deletar a VM com o comando:
```shell
pulumi destroy 
```

### Objetivo #4 Criando varias instancias em um unico arquivo.

Chegamos ao arquivo final.

Criaremos duas novas listas:
```python
instances_names = ["web1","web2", "web3"] # Array com nomes das instancias
output_public_ip = []
```

Um for para percorrer a lista de nomes:
```python
for instance in instances_names:
    ec2_instance = ec2.Instance(instance, # mude o nome para variavel
                            ami="ami-053b0d53c279acc90",
                            instance_type='t3.nano',
                            vpc_security_group_ids=[sg.id],
                            tags={
                                'Name': instance # mude o nome para variavel
                            })
    output_public_ip.append(ec2_instance.public_ip) # Popularemos o array com ip publico de cada instancia
```

Vamos mudar o export para:
```python
pulumi.export('public_ip', output_public_ip)
pulumi.export('instance_url', pulumi.Output.concat("http://", ec2_instance.public_dns ))
```

Assim voce pode executar o comando:
```shell
pulumi up
```

E veja a magica acontecer, guelo.
