# Desafio Técnico AutoLabs - Gestor Júnior de Automações

Este repositório contém a solução desenvolvida para o desafio técnico de automação e qualificação de leads da Clínica Renova. 

O fluxo foi construído utilizando o **n8n** para orquestrar o recebimento dos dados, **Google Gemini** para inteligência artificial e análise cognitiva, **Supabase** para armazenamento em banco de dados e **webhook.site** para simulação de disparo de mensagens.

## O que o fluxo faz
1. **Recepção:** Recebe os dados de um lead através de um Webhook (método POST).
2. **Processamento com IA:** Envia os dados do lead e as regras de negócio para o LLM classificar a oportunidade (QUENTE, MORNO ou FRIO) e redigir uma mensagem personalizada.
3. **Tratamento e Persistência:** Faz o parse do JSON retornado pela IA e salva o registro completo (dados do lead + classificação) em uma tabela no Supabase.
4. **Roteamento Inteligente:** Um nó condicional (Switch) avalia a classificação. Leads quentes e mornos disparam um HTTP Request simulando o envio de uma mensagem de WhatsApp. Leads frios pulam essa etapa.
5. **Retorno:** Devolve um JSON estruturado como resposta da requisição original, confirmando o fim do fluxo.

## Decisões Técnicas
* **Google Gemini como LLM:** Optei por utilizar o modelo `gemini-3-flash-preview` em vez da OpenAI ou Claude. A escolha se deu pela velocidade e precisão do modelo em estruturar saídas estritamente em `JSON Object`, somado ao excelente custo-benefício.
* **Tratamento de Dados pré-Supabase:** Para evitar incompatibilidades de tipo e erros de servidor (HTTP 500) no momento do `Create`, adicionei um nó `Edit Fields` após a IA. Ele utiliza a função `JSON.parse` para desmembrar a resposta em texto da IA e mapear perfeitamente as chaves para as colunas do banco de dados, permitindo a inserção via Auto-Map.
* **Lógica Condicional:** A segregação foi feita garantindo que apenas leads com intenção válida gerassem requisições no endpoint de disparo simulado, otimizando recursos e evitando abordagens indesejadas a leads sem qualificação ("Frio").

## Como testar o fluxo

1. Clone este repositório e baixe o arquivo `.json` do fluxo.
2. Importe o arquivo na sua instância do n8n.
3. Configure suas credenciais (Google Gemini API e Supabase API).
4. Ative o fluxo no canto superior direito do n8n.
5. Utilize o terminal para disparar uma requisição de teste. Substitua a URL abaixo pela URL do seu Webhook de produção gerada pelo n8n:

**Exemplo de Payload de Teste (Lead Quente):**
```bash
curl -X POST http://localhost:5678/webhook/receber-lead -H "Content-Type: application/json" -d '{"nome": "Carla Mendes", "empresa": "Própria Salão de Beleza", "segmento": "Beleza e Estética", "telefone": "71999887766", "interesse": "Harmonização facial", "como_conheceu": "Instagram", "ja_fez_procedimento": "Sim"}'
```

## Autor

Desenvolvido por Ricardo Durey Neto para o processo seletivo da AutoLabs.
