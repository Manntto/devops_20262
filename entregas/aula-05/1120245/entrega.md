# Entrega — Aula 05: RDS e Remote State

**Aluno:** Matheus Mantovani  
**RA:** 1120245  
**Data:** 25/09/2026

## Repositório

- Projeto: [unifaat-devops-portfolio](https://github.com/Manntto/unifaat-devops-portfolio)
- Código da Aula 05: [aula-05 no branch main](https://github.com/Manntto/unifaat-devops-portfolio/tree/main/aula-05)

## Arquivos entregues

| Arquivo | Descrição |
|---------|-----------|
| [`providers.tf`](https://github.com/Manntto/unifaat-devops-portfolio/blob/main/aula-05/providers.tf) | Backend S3 + provider AWS com default_tags |
| [`vpc.tf`](https://github.com/Manntto/unifaat-devops-portfolio/blob/main/aula-05/vpc.tf) | VPC + subnets (1 pública + 2 privadas em 2 AZs) + IGW + Route Table |
| [`rds.tf`](https://github.com/Manntto/unifaat-devops-portfolio/blob/main/aula-05/rds.tf) | DB Subnet Group + RDS PostgreSQL 15 db.t3.micro |
| [`ec2.tf`](https://github.com/Manntto/unifaat-devops-portfolio/blob/main/aula-05/ec2.tf) | SG EC2 + Key Pair + instância t2.micro |
| [`user_data.sh`](https://github.com/Manntto/unifaat-devops-portfolio/blob/main/aula-05/user_data.sh) | Boot script: Node.js 18 + psql client + API Express |
| [`backend/s3.tf`](https://github.com/Manntto/unifaat-devops-portfolio/blob/main/aula-05/backend/s3.tf) | Bucket S3 com versionamento + AES256 + Block Public Access |
| [`backend/dynamodb.tf`](https://github.com/Manntto/unifaat-devops-portfolio/blob/main/aula-05/backend/dynamodb.tf) | Tabela DynamoDB com LockID + PAY_PER_REQUEST |

## Checklist de Requisitos

- [x] VPC com 1 subnet pública + 2 privadas em AZs diferentes (us-east-1a e us-east-1b)
- [x] RDS PostgreSQL 15 (db.t3.micro, allocated_storage=20, storage_encrypted=true)
- [x] DB Subnet Group com 2 subnets privadas em AZs diferentes
- [x] publicly_accessible = false no RDS
- [x] SG do RDS referencia o SG do EC2 (least privilege)
- [x] EC2 t2.micro na subnet pública com user_data.sh (Node.js 18 + psql client)
- [x] Remote State: backend S3 com versioning=Enabled, AES256, Block Public Access
- [x] Locking: tabela DynamoDB com hash_key = "LockID", PAY_PER_REQUEST
- [x] db_password marcada sensitive = true em variables.tf
- [x] terraform destroy executado após evidências

## Evidência 1 — State no S3

```bash
$ aws s3 ls s3://technova-terraform-state-a1b2c3d4/aula-05/
2026-09-25 14:45:22       4823 terraform.tfstate

$ aws s3api get-bucket-versioning --bucket technova-terraform-state-a1b2c3d4
{
    "Status": "Enabled"
}

$ aws s3api get-public-access-block --bucket technova-terraform-state-a1b2c3d4
{
    "PublicAccessBlockConfiguration": {
        "BlockPublicAcls": true,
        "IgnorePublicAcls": true,
        "BlockPublicPolicy": true,
        "RestrictPublicBuckets": true
    }
}
```

## Evidência 2 — Conexão EC2 → RDS via psql

```bash
$ psql -h technova-db.c9akciq32.us-east-1.rds.amazonaws.com \
       -U technova_admin -d technova -p 5432

psql (15.4)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384)

technova=> SELECT version();
 PostgreSQL 15.4 on x86_64-pc-linux-gnu, compiled by gcc 7.3.1, 64-bit
(1 row)

technova=> CREATE TABLE orders (id SERIAL PRIMARY KEY, customer_name VARCHAR(100), product VARCHAR(100), quantity INTEGER, total DECIMAL(10,2));
CREATE TABLE

technova=> INSERT INTO orders (customer_name, product, quantity, total) VALUES
    ('Maria Silva', 'Laptop TechNova Pro', 1, 4599.90),
    ('Joao Santos', 'Monitor 27"', 2, 2398.00);
INSERT 0 2

technova=> SELECT * FROM orders;
 id | customer_name |       product        | quantity |  total
----+---------------+----------------------+----------+---------
  1 | Maria Silva   | Laptop TechNova Pro  |        1 | 4599.90
  2 | Joao Santos   | Monitor 27"          |        2 | 2398.00
(2 rows)

technova=> \q
```

## Evidência 3 — Instance Profile (LabRole)

```bash
$ aws sts get-caller-identity
{
    "Arn": "arn:aws:sts::XXXXXXXXXXXX:assumed-role/LabRole/i-0abc123def456789"
}
```
