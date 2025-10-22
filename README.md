# Projeto Node Endpoint Data Logger

Este projeto foi desenvolvido por **Eduardo Gomes** para registrar logs de acesso de forma automatizada utilizando **Node.js e Express**, simulando um ambiente real de auditoria e segurança da informação.

---

## 🧩 Objetivo do Projeto

Implementar um servidor HTTP simples que:
- Disponibilize um **endpoint de saúde** (`/ping`)
- Disponibilize um **endpoint de auditoria** (`/access-log`)
- Grave logs de acessos no diretório seguro `/var/log`
- Garanta persistência, formatação e registro de dados sensíveis de forma controlada

---

## 🏗️ Estrutura do Projeto

```
node-endpoint_data/
├── app.js
├── server.out
├── server.err
└── /var/log/xp_access.log
```

---

## ⚙️ Tecnologias Utilizadas

- **Node.js v20.19.5**
- **Express.js**
- **Linux Ubuntu Server (Azure VM)**
- **cURL** para testes
- **Permissões e logs do sistema (/var/log)**

---

## 📁 Conteúdo do app.js

```javascript
const express = require('express');
const fs = require('fs');
const path = '/var/log/xp_access.log';
const app = express();

app.use(express.json());

// Endpoint de verificação
app.get('/ping', (req, res) => res.status(200).send('pong'));

// Endpoint de gravação de logs
app.post('/access-log', (req, res) => {
  const entry = `${new Date().toISOString()} | ${req.ip} | ${JSON.stringify(req.body)}\n`;
  fs.appendFile(path, entry, (err) => {
    if (err) {
      console.error('ERRO ao gravar log:', err);
      return res.status(500).send('Erro ao gravar log');
    }
    console.log('Log gravado:', entry.trim());
    return res.status(201).send('Log gravado com sucesso');
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Servidor rodando na porta ${PORT}`));
```

---

## 🧰 Passo a Passo da Execução

### 1️⃣ Criar o arquivo de log e definir permissões
```bash
sudo touch /var/log/xp_access.log
sudo chmod 666 /var/log/xp_access.log
```

### 2️⃣ Instalar dependências
```bash
npm init -y
npm install express
```

### 3️⃣ Executar o servidor
```bash
node ~/node-endpoint_data/app.js > ~/node-endpoint_data/server.out 2> ~/node-endpoint_data/server.err &
sleep 1
tail -n +1 ~/node-endpoint_data/server.out
```
Saída esperada:
```
Servidor rodando na porta 3000
```

### 4️⃣ Testar o endpoint de saúde
```bash
curl -i http://localhost:3000/ping
```
Saída esperada:
```
HTTP/1.1 200 OK
pong
```

### 5️⃣ Enviar requisição POST para o endpoint de log
```bash
curl -i -X POST http://localhost:3000/access-log   -H "Content-Type: application/json"   -d '{"usuario":"eduardo","acao":"teste"}'
```
Saída esperada:
```
HTTP/1.1 201 Created
Log gravado com sucesso
```

### 6️⃣ Verificar logs gravados
```bash
sudo tail -n 5 /var/log/xp_access.log
```
Exemplo de saída:
```
2025-10-22T21:02:54.532Z | ::ffff:127.0.0.1 | {"usuario":"eduardo","acao":"teste"}
```

---

## 🧠 Diagnóstico de Erros Corrigidos

Durante o desenvolvimento, foram encontradas e resolvidas as seguintes situações:

- ❌ Porta 3000 já estava sendo usada por um servidor Vite (frontend React)  
  ➜ Solução: identificado com `ps -fp <PID>` e finalizado via `sudo kill -9 <PID>`  
- ❌ Erro 404 Not Found nos testes iniciais  
  ➜ Causa: servidor incorreto estava ativo na mesma porta  
  ➜ Solução: reinício do servidor Express com o `app.js` correto  
- ✅ Após correções, o servidor respondeu corretamente ao `curl` e gerou logs no `/var/log`.

---

## 🧾 Teste Final Realizado

```
curl -i -X POST http://localhost:3000/access-log   -H "Content-Type: application/json"   -d '{"usuario":"eduardo","acao":"teste"}'
```
Saída:
```
HTTP/1.1 201 Created
Log gravado com sucesso
```

Verificação de log:
```
sudo tail -n 5 /var/log/xp_access.log
2025-10-22T21:02:54.532Z | ::ffff:127.0.0.1 | {"usuario":"eduardo","acao":"teste"}
```

---

## 🧾 Log de Execuções Anteriores

Para registro e auditoria, as execuções do script são registradas em `/var/log/anonymize.log` e `/var/log/xp_access.log`, demonstrando o controle e histórico de execução de scripts automatizados.

---

## 👨‍💻 Autor

**Eduardo Gomes**  
Gerente de TI e estudante de Engenharia de Software  
Disciplina: **Cybersecurity - FIAP (Logs, Auditoria e Segurança da Informação)**  
Data de conclusão: **22 de outubro de 2025**
