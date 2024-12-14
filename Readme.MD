# Desenvolvimento de Website para Marcação de Consultas Dermatológicas

## Introdução

O projeto consiste no desenvolvimento de um **website** para marcação de consultas dermatológicas da **Skincare e Estética**, com o objetivo de facilitar o agendamento tanto para pacientes quanto para os profissionais da clínica. O website permite que os pacientes escolham a especialidade desejada, consultem a disponibilidade e agendem consultas de forma rápida e eficiente.

O **website** foi criado de forma simples e prática, para que qualquer pessoa consiga usá-lo sem dificuldades. O objetivo é permitir o acesso fácil, por meio de aplicativos de pesquisa, oferecendo bem-estar aos clientes de forma simples e rápida.

Ao entrar na nossa página, o cliente pode fazer um **cadastro gratuito**, o que nos ajuda a reconhecê-lo em visitas futuras. Depois disso, ele pode escolher o procedimento desejado, visualizando os horários disponíveis e os preços. Também temos pacotes promocionais exclusivos para novos clientes. Quando ele terminar a escolha, aparecerá uma confirmação com a data e o horário da consulta. Caso haja qualquer problema, temos um link que leva ao nosso **WhatsApp**, onde nossos atendentes estão prontos para ajudar e responder a qualquer dúvida.

### Fluxo de Agendamento

1. O usuário começa preenchendo um formulário com informações pessoais, como nome, e-mail e telefone, e confirma o agendamento.
2. Depois, acessa a página inicial, onde pode ver informações sobre os serviços e valores.
3. Ao escolher um serviço estético, ele clica em "Agendar" e é direcionado a um calendário que mostra as datas e horários disponíveis.
4. O pagamento será feito de forma presencial após o procedimento, aceitando cartão de crédito, débito, Pix ou dinheiro.

O **website** facilita o agendamento de consultas, permitindo que os pacientes encontrem o profissional mais adequado para suas necessidades. Além disso, o agendamento online permite que a clínica envie lembretes por **e-mail** ou **WhatsApp**, evitando esquecimentos e tornando o processo mais ágil.

---

## Código Backend para Marcação de Consultas

O seguinte código utiliza o **Express** para criar um servidor e o **PostgreSQL** para armazenar as informações de pacientes e consultas.

### Dependências

- **express**: Framework web para Node.js.
- **pg**: Cliente PostgreSQL para Node.js.
- **cors**: Middleware para habilitar CORS (Cross-Origin Resource Sharing).

### Código

```javascript
// Importa os módulos necessários
const express = require('express');
const { Pool } = require('pg');
const cors = require('cors');

// Configuração do pool de conexões PostgreSQL
const pool = new Pool({
    user: 'postgresaula',
    host: 'postgres-aula.cuebxlhckhcy.us-east-1.rds.amazonaws.com',
    database: 'postgresaula',
    password: 'PostgresAula123!',
    port: 5432,
    ssl: { rejectUnauthorized: false }, // Configuração de SSL, se necessário
});

// Configuração do servidor Express
const app = express();
app.use(cors()); // Habilita CORS para permitir requisições de outros domínios
app.use(express.urlencoded({ extended: true })); // Para processar dados de formulários
app.use(express.json()); // Para processar dados JSON

// Função para inserir dados na tabela "pacientes_lucas"
async function inserirPacientes({ nome, email, telefone }) {
    const query = `
        INSERT INTO pacientes_lucas (nome_completo, email, telefone)
        VALUES ($1, $2, $3)
        RETURNING id;
    `;
    const values = [nome, email, telefone];

    const result = await pool.query(query, values);
    return result.rows[0].id; // Retorna o ID do paciente inserido
}

// Função para inserir dados na tabela "consulta_lucas"
async function inserirConsulta({ pacienteId, data, hora, motivo }) {
    const query = `
        INSERT INTO consulta_lucas (paciente_id, data_consulta, hora_consulta, motivo_consulta)
        VALUES ($1, $2, $3, $4);
    `;
    const values = [pacienteId, data, hora, motivo];

    await pool.query(query, values);
}

// Rota para receber dados do formulário e inserir no banco
app.post('/submit', async (req, res) => {
    console.log('Dados recebidos:', req.body); // Log dos dados recebidos

    try {
        const { nome, email, telefone, data, hora, motivo } = req.body;

        // Inserir os dados do paciente
        const pacienteId = await inserirPacientes({ nome, email, telefone });

        // Inserir os dados da consulta
        await inserirConsulta({ pacienteId, data, hora, motivo });

        res.status(200).send('Dados inseridos com sucesso!');
    } catch (error) {
        console.error('Erro ao processar requisição:', error);
        res.status(500).send('Erro ao inserir dados: ' + error.message);
    }
});

// Rota para testar o servidor
app.get('/', (req, res) => {
    res.send('Servidor funcionando!');
});

// Inicia o servidor
const PORT = 3000;
app.listen(PORT, () => {
    console.log(`Servidor rodando na porta ${PORT}`);
});
