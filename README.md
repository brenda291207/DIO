# DIO
Calculadora de Emissão de Gás Carbônico
# 🍃 Calculadora de Emissão de CO₂

Projeto simples em HTML/CSS/JS que calcula emissões aproximadas de CO₂ para um trajeto com base na distância e modo de transporte.

Estrutura:
- index.html
- css/style.css
- js/routes-data.js
- js/config.js
- js/calculator.js
- js/ui.js
- js/app.js

Instruções para criar repositório local e enviar ao GitHub:

1. Crie o repositório no GitHub (ex.: `calculadora-co2`) ou use o nome desejado.

2. No terminal, dentro da pasta do projeto:
```bash
git init
git add .
git commit -m "Initial commit: CO2 calculator"
git branch -M main
# Substitua a URL abaixo pela URL do seu repositório no GitHub
git remote add origin git@github.com:SEU_USUARIO/NOME_DO_REPO.git
git push -u origin main
```

Se preferir HTTPS:
```bash
git remote add origin https://github.com/SEU_USUARIO/NOME_DO_REPO.git
git push -u origin main
```

3. (Opcional) Habilitar GitHub Pages:
- Vá em Settings > Pages, escolha a branch `main` e root (`/`) e salve.
- O site ficará disponível em https://SEU_USUARIO.github.io/NOME_DO_REPO

Notas:
- `js/routes-data.js` usa dados locais (exemplos) para preencher o datalist e retornar distâncias aproximadas.
- Ajuste `js/config.js` para trocar fatores de emissão por valores oficiais ou atualizados.
- A interação visual dos modos de transporte está em `js/ui.js`.
- As seções de resultados começam ocultas (.hidden) e são mostradas ao submeter o formulário.
