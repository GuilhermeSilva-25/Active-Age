# 🗄️ Modelagem de Dados - Active Age

A arquitetura de dados do Active Age suporta a jornada clínica completa do paciente idoso, gerenciando transações financeiras, agendamentos e registros imutáveis de saúde.

## 🧩 Entidades e Coleções Principais

### 1. Usuarios
Armazena credenciais, tokens e perfis de acesso. Utiliza herança para diferenciar os perfis:
* `PacienteCuidador`: Possui métodos para upload de exames antigos e download de documentos médicos.
* `MedicoGeriatra`: Possui atributos profissionais (CRM), `statusAssinatura` (Ativa, Inadimplente) e métodos para registrar prontuário, emitir receitas, pedidos de exames e atestados.
* `Administrador`: Contém métodos de moderação de perfis e visualização de dashboards gerenciais.

### 2. Agendamentos
Gerencia o elo entre o paciente, o médico e o fluxo financeiro.
* **Relacionamento:** 1 Paciente -> N Agendamentos; 1 Médico -> N Agendamentos.
* **Atributos Chave:** `dataHora`, `status` (Aguardando Pagamento, Confirmado, Cancelado, Concluído), `linkTeleconsulta` e `valorConsulta`.

### 3. PagamentosTransacoes
Gerencia o fluxo financeiro gerado pelo Gateway de Pagamentos externo.
* **Relacionamento:** Vinculado a um Usuário (Paciente pagando consulta ou Médico pagando assinatura).
* **Atributos Chave:** `valor`, `dataPagamento`, `statusPagamento` (Aprovado, Pendente, Recusado), `tipoPagamento` (Consulta, Assinatura_Mensal) e `gatewayId` (Referência da transação externa).

### 4. ProntuariosEletronicos (O Coração Clínico)
Registra a evolução clínica (queixas, diagnóstico, conduta) feita pelo médico.
* **Relacionamento:** 1 Agendamento -> 1 Prontuário (Relação 1:1).
* **Regra de Segurança:** Possui um atributo booleano crítico `imutavel`. Após a consulta ser finalizada e assinada, as informações clínicas centrais não podem ser alteradas.

### 5. DocumentosMedicosDigitais
Gerados a partir do Prontuário Eletrônico, contemplam todos os papéis legais que o médico entrega ao paciente em PDF. Podem ser classificados em subtipos:
* `Receita`: Medicamentos e posologia.
* `Atestado`: Justificativas de repouso ou declarações de saúde.
* `PedidoDeExame`: Solicitações de análises clínicas ou de imagem.

### 6. DocumentosPaciente (Histórico Prévio)
Gerencia arquivos e exames antigos enviados voluntariamente pelo paciente/cuidador antes da consulta.
* **Atributos Chave:** `urlArquivo` (armazenamento externo/GridFS), `dataUpload`, `tipoDocumento`. O médico possui permissão de leitura restrita durante a consulta.