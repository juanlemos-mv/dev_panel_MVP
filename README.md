# Dev Panel (MVP)

## Objetivo
Aplicação local para desenvolvedores gerenciarem projetos com arquitetura estilo **libs + backend**:
build seletivo de módulos, subida de processos (backend/agent/front), perfis de variáveis de ambiente,
templates de execução e visualização de logs em tempo real.

O foco inicial é suportar projetos no padrão do **Fatureasy**, mas com um design extensível para aceitar
outros projetos que sigam o mesmo estilo (múltiplos módulos + backend expondo libs).

## Arquitetura do Projeto
O Dev Panel é dividido em 3 partes (mesmo “jeito” do Fatureasy):

### 1) `libs/` (inteligência)
Contém a lógica de domínio e integração. O backend não conhece detalhes de execução; só expõe.

Exemplos de responsabilidades por lib:
- **lib-project**: identificar e descrever um projeto (paths, estrutura, tipo).
- **lib-templates**: perfis de ambiente e templates de execução.
- **lib-executor**: executar comandos (ProcessBuilder) com env isolado.
- **lib-logs**: capturar stdout/stderr, persistir e streamar (SSE/WebSocket).
- **lib-integrations**: “adapters/plugins” por tipo de projeto (ex.: FatureasyIntegration).

### 2) `backend/` (exposição)
Aplicação Spring Boot responsável por:
- expor as libs via REST (e SSE para logs),
- mapear Beans (@Configuration / @Bean),
- controlar autorização local (se necessário),
- persistência via PostgreSQL.

**Regra:** backend não implementa regra complexa; delega para libs.

### 3) `frontend/` (painel)
HTML/CSS/JS simples voltado para devs:
- seleção de projeto e perfil,
- botões para ações (build/run/stop),
- console de logs ao vivo.

## Extensibilidade (integrações)
O sistema suporta “tipos de projeto” através de integrações.

Uma integração define:
- como detectar o projeto (ex.: ler pom, módulos),
- quais ações existem (build libs, run backend, run agent, etc.),
- quais variáveis padrão e templates sugeridos,
- como montar comandos (mvn -pl, npm start, etc.).

MVP: **Fatureasy-style integration** (root + `libs/pom.xml` + `backend/pom.xml`).

## Stack
- Java 21 + Spring Boot 3 (backend)
- PostgreSQL (persistência)
- Execução: ProcessBuilder (processos do SO com env injetado)
- Logs ao vivo: SSE (Server-Sent Events)
- Front: HTML/CSS/JS

## Escopo do MVP
- Cadastro de projetos e perfis (env vars).
- Detecção do tipo de projeto (Fatureasy-style).
- Ações:
  - Build libs seletivo (`-pl` + `-am` + `-DskipTests`)
  - Run backend (spring-boot:run com args)
  - Run agent
  - Run frontend
- Logs ao vivo + histórico de execuções.
