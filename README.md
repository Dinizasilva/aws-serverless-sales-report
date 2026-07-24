<img src="IMG_0295%20(1).jpg" width="500" alt="Banner do Projeto">

🚀 Automação Serverless de Relatórios de Vendas com AWS Lambda
Repositório dedicado à documentação de um laboratório prático de arquitetura serverless na AWS, focado na automação de extração de dados de um banco MySQL e envio de relatórios via Amazon SNS.

🛠️ Arquitetura e Tecnologias Utilizadas
AWS Lambda: Execução de código em Python (3.9) sem gerenciar servidores.

AWS IAM: Gestão de permissões e papéis de execução seguros (salesAnalysisReportRole e salesAnalysisReportDERole).

AWS Lambda Layers: Reutilização de dependências externas (PyMySQL).

Amazon VPC & Security Groups: Conexão segura das funções Lambda com instâncias EC2 rodando o banco MySQL da cafeteria.

Amazon SNS (Simple Notification Service): Mensageria e publicação de alertas/relatórios por e-mail.

Amazon EventBridge (CloudWatch Events): Orquestração e agendamento de execuções via expressões Cron.

AWS CLI: Implantação e gerenciamento de recursos via linha de comando.

📝 Passo a Passo da Execução e Resolução de Problemas
1. Configuração e Análise de Perfis do IAM
Ação: Validação dos papéis de execução no IAM para garantir os privilégios mínimos necessários:

salesAnalysisReportRole: Configurado com políticas para acesso total ao SNS (AmazonSNSFullAccess), leitura do Systems Manager (AmazonSSMReadOnlyAccess), logs do CloudWatch (AWSLambdaBasicRunRole) e permissão para invocar outras Lambdas (AWSLambdaRole).

salesAnalysisReportDERole: Configurado com permissões básicas de logs (AWSLambdaBasicRunRole) e acesso à VPC (AWSLambdaVPCAccessRunRole).

2. Criação de Lambda Layer e Extração de Dados
Ação: Criação de uma Lambda Layer chamada pymysqlLibrary fazendo o upload do pacote .zip contendo a biblioteca de cliente PyMySQL para comunicação com o MySQL.

Criação da Função: Implementação da função extratora salesAnalysisReportDataExtractor, associando-a à camada e configurando o manipulador (handler) em Python.

Configuração de Rede (VPC): Associação da função à VPC da cafeteria, subnet pública e ao CafeSecurityGroup para permitir o acesso aos dados do banco.

3. Resolução de Problemas: Erro de Timeout no Banco de Dados
Problema Encontrado: Ao realizar o primeiro teste na função extratora (SARDETestEvent), a execução falhou com a seguinte mensagem de erro:

{
  "errorMessage": "Task timed out after 3.00 seconds"
}

Diagnóstico e Solução:

O tempo limite padrão do Lambda de 3 segundos esgotou porque a função não estava conseguindo se comunicar com o banco de dados na instância EC2.

Causa raiz: As regras de entrada (Inbound Rules) do grupo de segurança do banco (CafeSecurityGroup) não liberavam a porta padrão do MySQL (3306).

Correção: Ajuste nas regras de entrada do grupo de segurança para permitir conexões na porta 3306, seguida pela população de dados reais de pedidos ao acessar o site da cafeteria hospedado na instância EC2 (/cafe). Após popular o banco, o teste retornou com sucesso (statusCode: 200).

4. Configuração de Notificações com Amazon SNS
Ação: Criação de um tópico SNS (salesAnalysisReportTopic) e inscrição de um endereço de e-mail corporativo/pessoal para receber os relatórios diários, validando a subscrição por confirmação de link enviado por e-mail.

5. Orquestração, AWS CLI e Gatilhos Agendados
Ação na CLI: Conexão à instância de terminal via EC2 Instance Connect, configuração das credenciais com aws configure e criação da função principal de relatório salesAnalysisReport utilizando o comando aws lambda create-function.

Variáveis de Ambiente: Definição da variável topicARN na função principal para apontar para o tópico do SNS criado anteriormente.

Automação com EventBridge: Adição de um gatilho do tipo EventBridge (CloudWatch Events) configurado com uma expressão Cron (cron(15 4 ? * MON-SAT *)) para disparar o relatório automaticamente de segunda a sábado em horários programados.

💡 Como Executar / Testar
Certifique-se de que o banco de dados MySQL na instância EC2 esteja ativo e populado com pedidos da cafeteria.

Acesse o console da AWS Lambda e realize testes manuais nas funções salesAnalysisReportDataExtractor e salesAnalysisReport.

Verifique a caixa de entrada do e-mail cadastrado no Amazon SNS para visualizar o relatório diário formatado.
