# Exercício Prático: Modelagem e Consultas em MongoDB

---

## Parte 1 — Modelagem de Dados

### 1.1 — Incorporação: curso com módulos embutidos

```javascript
db.cursos.insertOne({
  "_id": ObjectId(),
  "titulo": "Desenvolvimento Web Full Stack",
  "descricao": "Curso completo de desenvolvimento web",
  "preco": 299.90,
  "modulos": [
    { "nome": "Introdução ao HTML e CSS",  "carga_horaria": 10 },
    { "nome": "JavaScript Essencial",      "carga_horaria": 20 },
    { "nome": "Node.js e APIs REST",       "carga_horaria": 15 },
    { "nome": "Banco de Dados com MongoDB","carga_horaria": 12 }
  ]
})
```

**Por que incorporar?**
Os módulos pertencem exclusivamente ao curso e raramente são consultados de forma isolada. Ao incorporá-los, uma única leitura retorna o documento completo — sem necessidade de joins ou consultas adicionais — o que reduz a latência e melhora o desempenho de leitura.

---

### 1.2 — Referência: instrutor e curso separados

```javascript
db.instrutores.insertOne({
  "_id": ObjectId("64a1b2c3d4e5f6a7b8c9d001"),
  "nome":  "Ana Souza",
  "email": "ana.souza@plataforma.com",
  "bio":   "Engenheira de software com 10 anos de experiência"
})

db.cursos.insertOne({
  "_id":          ObjectId(),
  "titulo":       "Python para Data Science",
  "preco":        349.90,
  "instrutor_id": ObjectId("64a1b2c3d4e5f6a7b8c9d001")
})
```

**Quando preferir referência?**
Quando o instrutor ministra vários cursos. Se os dados dele fossem copiados em cada curso, uma simples atualização de bio ou e-mail exigiria alterar dezenas de documentos. Com a referência, basta atualizar um único documento na coleção `instrutores`.

---

## Parte 2 — Consultas e Filtros

### 2.1 — Filtro de existência ($exists)

```javascript
db.clientes.find({
  "email": { $exists: true }
})
```

### 2.2 — Filtro de comparação ($gte)

```javascript
db.clientes.find({
  "idade": { $gte: 21 }
})
```

### 2.3 — Filtro complexo (cidade E idade)

```javascript
db.clientes.find({
  "cidade": "São Paulo",
  "idade":  { $lt: 30 }
})
```

---

## Parte 3 — Pipeline de Agregação

### 3.1 — Relatório de vendas concluídas por categoria

```javascript
db.vendas.aggregate([
  {
    $match: {
      "status": "concluída"
    }
  },
  {
    $group: {
      "_id":              "$categoria",
      "total_quantidade": { $sum: "$quantidade" }
    }
  },
  {
    $project: {
      "_id":              0,
      "categoria":        "$_id",
      "total_quantidade": 1
    }
  }
])
```

**Lógica do pipeline:**
- **$match** — filtra somente vendas com status "concluída", descartando as demais logo no início para reduzir o volume processado.
- **$group** — agrupa os documentos por categoria e soma o campo `quantidade` de cada grupo.
- **$project** — renomeia `_id` para `categoria` e oculta o campo original, deixando o resultado mais legível.

