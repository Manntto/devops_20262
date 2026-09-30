# Entrega — Prova do Primeiro Bimestre (DevOps)

**Aluno:** Matheus Mantovani  
**RA:** 1120245  
**Data:** [preencher no dia da prova]  
**Ferramenta de IA utilizada:** Kiro (Spec-Driven Development)

## Repositório do Projeto

- URL: https://github.com/Manntto/prova-primeiro-bimestre-devops

## Checklist de Evidências

- [x] Repositório público com README (nome + RA) e .gitignore
- [x] Mínimo de 6 commits com Conventional Commits + feature branch
- [x] API com **CRUD completo** de reservas (POST, GET, GET/:id, PUT, DELETE) + /health
- [x] Rotas de CRUD gravando no **banco PostgreSQL** (não em memória)
- [x] Dockerfile funcional da API de Reservas
- [x] docker-compose.yml (API + PostgreSQL) subindo com um comando
- [x] Terraform modularizado (vpc, security-group, ec2, rds)
- [x] **RDS PostgreSQL provisionado** nas subnets privadas (banco da API na nuvem)
- [x] Remote State configurado (S3 + DynamoDB)
- [x] Uso de LabRole/LabInstanceProfile (sem criar IAM próprio)
- [x] terraform validate e terraform plan sem erros
- [x] relatorio.md completo (4 questões)
- [ ] terraform destroy executado após evidências

## Evidências

### Docker Build
```
[+] Building 12.8s (13/13) FINISHED
 => [builder 4/4] RUN npm install --omit=dev
 => [stage-1 6/6] RUN addgroup -S appgroup && adduser -S appuser -G appgroup
 => naming to docker.io/library/api-reservas:1.0
```

### Docker Compose PS
```
NAME           IMAGE                                STATUS
reservas-api   prova-primeiro-bimestre-devops-api   Up 2 minutes
reservas-db    postgres:15-alpine                   Up 2 minutes (healthy)
```

### Terraform Plan
```
Plan: 14 to add, 0 to change, 0 to destroy.
```

### Terraform Apply — Outputs
```
api_url        = "http://98.81.51.238:3000"
ec2_public_ip  = "98.81.51.238"
ec2_public_dns = "ec2-98-81-51-238.compute-1.amazonaws.com"
rds_endpoint   = "reservas-rds.ceiks7fjab1o.us-east-1.rds.amazonaws.com:5432"
rds_host       = "reservas-rds.ceiks7fjab1o.us-east-1.rds.amazonaws.com"
```

### API funcionando na AWS (RDS)
```
GET  /health                → {"status":"ok"}
POST /reservas              → {"id":1,"cliente":"Matheus Mantovani","data":"2026-10-01","status":"confirmada"}
GET  /reservas              → [{"id":1,...},{"id":2,...}]
GET  /reservas/1            → {"id":1,"cliente":"Matheus Mantovani",...}
PUT  /reservas/2            → {"id":2,...,"status":"confirmada"}
DELETE /reservas/1          → {"mensagem":"Removida.",...}
GET  /reservas/1 (pós delete) → {"erro":"Reserva nao encontrada."} HTTP 404
```
