# Desenvolvimento de Website para Marcação de Consultas Dermatológicas

## Introdução

O projeto consiste no desenvolvimento de um **website** para marcação de consultas dermatológicas da **Skincare e Estética**, com o objetivo de facilitar o agendamento tanto para pacientes quanto para os profissionais da clínica. O website permite que os pacientes escolham a especialidade desejada, consultem a disponibilidade e agendem consultas de forma rápida e eficiente.

## Objetivos do Website

O **website** foi criado de forma simples e prática, para que qualquer pessoa consiga utilizá-lo sem dificuldades. A ideia é permitir o acesso fácil, através de aplicativos de pesquisa, oferecendo bem-estar aos nossos clientes de forma simples e rápida.

### Funcionalidades Principais:
- **Cadastro gratuito:** Ao acessar o site, o cliente pode realizar um cadastro, o que ajuda a reconhecê-lo em visitas futuras.
- **Escolha de procedimentos:** O paciente pode escolher o procedimento desejado e visualizar os horários disponíveis e os preços.
- **Pacotes promocionais:** Oferecemos pacotes promocionais exclusivos para novos clientes.
- **Confirmação de agendamento:** Após escolher o serviço, o paciente recebe uma confirmação com a data e o horário da consulta.
- **Suporte via WhatsApp:** Caso haja qualquer problema, o site conta com um link que direciona para o WhatsApp, onde os atendentes estão disponíveis para ajudar e esclarecer dúvidas.

## Fluxo de Agendamento

O processo de agendamento no site é bem simples e intuitivo, dividido em etapas:

1. **Preenchimento de formulário:** O usuário preenche um formulário com informações pessoais, como nome, e-mail e telefone.
2. **Escolha do serviço:** Depois de se cadastrar, o paciente acessa a página inicial, onde pode ver informações sobre os serviços e valores. Ao escolher um serviço estético, ele clica em "Agendar".
3. **Calendário de agendamento:** O paciente é redirecionado para um calendário onde pode visualizar as datas e horários disponíveis.
4. **Confirmação do agendamento:** Após selecionar o horário, o paciente confirma o agendamento e recebe uma notificação.
5. **Pagamento presencial:** O pagamento será feito presencialmente após o procedimento, aceitando cartão de crédito, débito, Pix ou dinheiro.

## Benefícios do Website

- **Facilidade no agendamento:** O website facilita muito a vida dos pacientes e profissionais envolvidos, permitindo agendar consultas de forma rápida e simples.
- **Lembretes automáticos:** O sistema envia lembretes automáticos por e-mail ou WhatsApp, ajudando a evitar esquecimentos e tornando o processo mais ágil.
- **Acesso fácil ao profissional adequado:** O paciente pode encontrar o profissional mais adequado para suas necessidades de forma rápida e eficiente.

## Conclusão

Criamos um **website** para o agendamento de consultas na clínica dermatológica **Skincare e Estética**, facilitando a vida dos pacientes e dos profissionais. O sistema de agendamento online proporciona conveniência, agilidade e uma experiência mais fluida para os clientes, além de otimizar a gestão da clínica.

# Código de Backend para Inserção de Dados em PostgreSQL

Este código utiliza **Express** para criar um servidor e **PostgreSQL** para manipular dados de pacientes e consultas. Ele permite inserir informações de pacientes e consultas em uma tabela no banco de dados PostgreSQL.

## Dependências

- **express**: Framework web para Node.js.
- **pg**: Cliente PostgreSQL para Node.js.
- **cors**: Middleware para habilitar o CORS (Cross-Origin Resource Sharing).

## Código

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
    res.send('Servidor funcionando222!');
});

// Inicia o servidor
const PORT = 3000;
app.listen(PORT, () => {
    console.log(`Servidor rodando na porta ${PORT}`);
});
