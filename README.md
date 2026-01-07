# IAGen_web3
Desenvolvendo IA Generativa &amp; Web3

O objetivo deste repositório é versionar os projetos de referência para desenvolvimento de agentes de IA e integrações: 

Abaixo, detalho a especificação técnica e funcional dividida por módulos.

1. Arquitetura Geral do Sistema
A solução será composta por três microsserviços principais desenvolvidos em Python (FastAPI):

Módulo de Gateway (Integração Uzap): Gerencia o recebimento de webhooks e o envio de mensagens.

Módulo de Cérebro (IA & RAG): Processa a lógica da IA, consulta o banco vetorial e gera a resposta.

Módulo de Ingestão (Base de Conhecimento): Pipeline de alimentação e atualização dos dados.

2. Especificação dos Módulos
Módulo A: Integração Uzap (Gateway)
Este módulo é a "pele" do projeto. Ele expõe um endpoint para a Uzap e formata a saída para o usuário.

Tecnologias: FastAPI, httpx (para requests assíncronos).

Processo Funcional:

Receber o JSON da Uzap via Webhook.

Validar o token da instância.

Extrair o chatId (telefone do usuário) e o text.

Enviar o texto para o Módulo de Cérebro.

Retornar a resposta via POST para o endpoint de envio da Uzap (/message/text).

Módulo B: O Cérebro (IA Generativa)
Aqui ocorre a mágica. Em vez de enviar a pergunta direto para o GPT-4 ou Claude, fazemos uma busca semântica.

Tecnologias: LangChain ou LlamaIndex, OpenAI API (ou local como Llama 3 via Ollama), ChromaDB ou Pinecone (Banco Vetorial).

Fluxo de Resposta:

Vetorização: A pergunta do usuário é convertida em um vetor (embedding).

Recuperação: O sistema busca no Banco Vetorial os 3 trechos de documentos mais similares.

Prompt Augmentation: Montamos um prompt: "Com base nos fatos: {contexto}, responda ao usuário: {pergunta}".

Geração: A IA gera a resposta final baseada estritamente no conhecimento fornecido.

Módulo C: Pipeline de Alimentação (Base de Conhecimento)
Este é o processo de "estudo" da IA. Ele deve ser capaz de ler PDFs, DOCX ou manuais do site da empresa.

Divisão de Processos de Alimentação:

Carregamento: Script Python que monitora uma pasta ou URL.

Fragmentação (Chunking): Divisão de textos longos em pedaços de ~1000 caracteres para não estourar a janela de contexto da IA.

Indexação: Conversão desses pedaços em vetores e salvamento no banco de dados.

3. Divisão do Projeto (Estrutura de Pastas)
Bash

projeto-ia-uzap/
├── app/
│   ├── gateway/            # Conexão com API Uzap
│   │   ├── routes.py       # Endpoints de Webhook
│   │   └── client.py       # Funções de envio (POST /message/text)
│   ├── brain/              # Lógica de RAG e IA
│   │   ├── chains.py       # Orquestração LangChain
│   │   └── prompts.py      # Templates de instruções da IA
│   └── ingestion/          # Alimentação da base
│       ├── loader.py       # Leitor de PDF/Texto
│       └── vector_store.py # Configuração do Banco Vetorial (ChromaDB)
├── data/                   # Documentos originais (PDFs/TXT)
├── .env                    # Chaves de API (Uzap Token, OpenAI Key)
└── requirements.txt        # Dependências (fastapi, langchain, openai, chromadb)


4. Requisitos de Manutenção da Base
Para garantir que a IA não forneça informações obsoletas:

Trigger de Atualização: Sempre que um arquivo na pasta /data for alterado, o loader.py deve limpar a coleção do banco vetorial referente àquele documento e re-indexar.

Feedback Loop: Armazenar logs de "Não sei responder" para identificar lacunas na base de conhecimento.
