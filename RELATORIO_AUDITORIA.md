# Relatório de Auditoria Técnica (verificação estática + execução em navegador)

Data: 2026-03-02
Escopo: `index.html` e `etiquetas-shopee.html`.

## Metodologia
- Validação estática de HTML com `htmlhint`.
- Verificação de sintaxe JavaScript dos blocos `<script type="module">` com `node --check`.
- Execução real em navegador (Playwright + `python -m http.server`) para capturar erros de carregamento e falhas de runtime.

## Achados principais

### 1) Recursos locais ausentes (quebra funcional/visual)
**Severidade:** Alta

- `etiquetas-shopee.html` referencia `css/styles.css`, porém o arquivo não existe no repositório atual. Isso gera `404` e impacta layout/estilo da tela.  
- `etiquetas-shopee.html` referencia `shared.js`, também inexistente, gerando `404` e potencial perda de funcionalidades globais (navbar/sidebar/utilitários compartilhados).  
- A execução de `index.html` também dispara tentativa de carregar `shared.js` e `css/styles.css` (via fluxo compartilhado), ambos com `404`.

**Evidências no código:**
- Referência ao CSS externo local em `etiquetas-shopee.html`.  
- Referência ao JS compartilhado em `etiquetas-shopee.html`.

### 2) Inconsistências de marcação HTML
**Severidade:** Média

- Em `index.html`, o placeholder do `textarea` está com aspas simples, contrariando regra de consistência de atributo em ferramenta de lint (pode não quebrar em navegador moderno, mas falha em padrão do lint).  
- O `htmlhint` sinalizou possível desequilíbrio de tags em `index.html` e `etiquetas-shopee.html`. Como o parser de navegador é permissivo, a página abre; mesmo assim, o alerta indica que há risco de manutenção e regressão em renderização futura.

**Evidências no código:**
- Placeholder com aspas simples em `index.html`.
- Trecho com fechamento comprimido de `div` em `etiquetas-shopee.html` próximo ao bloco TikTok.

### 3) Dependências externas críticas sem fallback
**Severidade:** Média

- O sistema depende de CDNs (`firebase`, `tailwind`, `font-awesome`, `pdf-lib`). Em ambiente offline/restrito, funcionalidades essenciais podem indisponibilizar.
- Não há fallback local para scripts/estilos críticos.

## Resultado dos checks executados

- `npx --yes htmlhint index.html etiquetas-shopee.html` → encontrou 4 erros (2 arquivos).
- `node --check` dos scripts `type="module"` extraídos de ambos arquivos → sem erro de sintaxe JS.
- Playwright + servidor local (`python3 -m http.server`) → confirmou `404` em `css/styles.css` e `shared.js`.

## Recomendações objetivas

1. **Restaurar/commitar os arquivos faltantes** (`shared.js` e `css/styles.css`) ou remover as referências caso não sejam mais usadas.
2. **Fechar pendências de lint HTML** (aspas duplas em atributos e revisão de estrutura de `divs`).
3. **Adicionar checklist de CI simples** para evitar regressão:
   - `htmlhint` (ou equivalente),
   - smoke test de abertura das páginas com validação de `requestfailed`/`404`.
4. **Opcional (resiliência):** manter cópia local mínima de dependências críticas ou fallback para modo degradado.
