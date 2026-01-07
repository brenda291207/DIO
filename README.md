# DIO
Calculadora de Emissão de Gás Carbônico
/* app.js
   Integra UI, RoutesData e Calculator para funcionamento básico.
*/
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('calculator-form');
  const originEl = document.getElementById('origin');
  const destEl = document.getElementById('destination');
  const distanceEl = document.getElementById('distance');
  const manualCheckbox = document.getElementById('manual-distance');

  const resultsSection = document.getElementById('results');
  const resultsContent = document.getElementById('results-content');
  const comparisonSection = document.getElementById('comparison');
  const comparisonContent = document.getElementById('comparison-content');
  const creditsSection = document.getElementById('carbon-credits');
  const creditsContent = document.getElementById('carbon-credits-content');

  function showSection(section) {
    section.classList.remove('hidden');
    section.setAttribute('aria-hidden', 'false');
  }

  function hideSection(section) {
    section.classList.add('hidden');
    section.setAttribute('aria-hidden', 'true');
  }

  function setDistanceFromRoutes() {
    const orig = originEl.value.trim();
    const dest = destEl.value.trim();
    if (!orig || !dest) return null;
    if (manualCheckbox.checked) return null;
    const d = window.RoutesData && window.RoutesData.getDistance ? window.RoutesData.getDistance(orig, dest) : null;
    if (d != null) {
      distanceEl.value = d;
      return d;
    }
    return null;
  }

  // toggle manual distance editing
  manualCheckbox.addEventListener('change', () => {
    if (manualCheckbox.checked) {
      distanceEl.readOnly = false;
      distanceEl.focus();
    } else {
      distanceEl.readOnly = true;
      // try to auto-fill when disabling manual
      setDistanceFromRoutes();
    }
  });

  // try update distance when origin/destination change (debounced minimal)
  let updateTimer = null;
  [originEl, destEl].forEach(el => {
    el.addEventListener('input', () => {
      if (updateTimer) clearTimeout(updateTimer);
      updateTimer = setTimeout(() => {
        setDistanceFromRoutes();
      }, 350);
    });
  });

  form.addEventListener('submit', (e) => {
    e.preventDefault();
    // determine distance
    let distance = Number(distanceEl.value);
    if (!manualCheckbox.checked) {
      const d = setDistanceFromRoutes();
      if (d != null) distance = Number(d);
    }

    if (!distance || distance <= 0 || isNaN(distance)) {
      resultsContent.innerHTML = '<p>Não foi possível determinar a distância. Insira manualmente ou escolha locais conhecidos.</p>';
      showSection(resultsSection);
      hideSection(comparisonSection);
      hideSection(creditsSection);
      return;
    }

    // get selected transport
    const transport = document.querySelector('input[name="transport"]:checked');
    const mode = transport ? transport.value : 'car';

    const kg = window.Calculator.calculateEmissionKg(distance, mode);
    const formatted = window.Calculator.formatKg(kg);

    // show results
    resultsContent.innerHTML = `
      <p><strong>Origem:</strong> ${escapeHtml(originEl.value || '—')}</p>
      <p><strong>Destino:</strong> ${escapeHtml(destEl.value || '—')}</p>
      <p><strong>Distância:</strong> ${distance.toFixed(1)} km</p>
      <p><strong>Transporte:</strong> ${mode}</p>
      <p><strong>Emissão estimada:</strong> ${formatted}</p>
    `;
    showSection(resultsSection);

    // comparison: emissions for all modes
    const factors = window.CO2_CONFIG && window.CO2_CONFIG.EMISSION_FACTORS ? window.CO2_CONFIG.EMISSION_FACTORS : {};
    const comparisonHtml = Object.keys(factors).map(m => {
      const val = window.Calculator.calculateEmissionKg(distance, m);
      return `<li><strong>${m}</strong>: ${window.Calculator.formatKg(val)}</li>`;
    }).join('');
    comparisonContent.innerHTML = `<ul>${comparisonHtml}</ul>`;
    showSection(comparisonSection);

    // carbon credits: simple conversion: 1 credit = 1 ton CO2
    const credits = kg / 1000; // tons
    creditsContent.innerHTML = `
      <p>Emissão total: <strong>${formatted}</strong></p>
      <p>Créditos de carbono necessários (1 crédito = 1 tonelada CO₂): <strong>${credits.toFixed(3)} créditos</strong></p>
    `;
    showSection(creditsSection);
  });

  // small helper to avoid XSS when injecting user values
  function escapeHtml(str) {
    if (!str) return '';
    return str.replace(/[&<>"']/g, function (s) {
      return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[s];
    });
  }
});
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
