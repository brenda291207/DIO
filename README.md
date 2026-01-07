<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>🍃 Calculadora de Emissão de CO₂</title>
  <link rel="stylesheet" href="css/style.css" />
</head>
<body class="calc">

  <header class="calc__header">
    <div class="calc__header-inner">
      <h1 class="calc__title">🍃 Calculadora de Emissão de CO₂</h1>
      <p class="calc__subtitle">Calcule as emissões de CO₂ estimadas para um trajeto e compare opções de transporte.</p>
    </div>
  </header>

  <main class="calc__main" id="main-content">
    <form id="calculator-form" class="calc__form" novalidate>
      <fieldset class="calc__fieldset calc__fieldset--route">
        <legend class="calc__legend">Trajeto</legend>

        <div class="calc__form-row">
          <label class="calc__label" for="origin">Origem</label>
          <input
            id="origin"
            name="origin"
            class="calc__input calc__input--text"
            type="text"
            list="cicles-list"
            placeholder="Digite a cidade ou local de origem"
            aria-required="true"
            autocomplete="off"
          />
          <datalist id="cicles-list">
            <!-- Populado via JavaScript -->
          </datalist>
        </div>

        <div class="calc__form-row">
          <label class="calc__label" for="destination">Destino</label>
          <input
            id="destination"
            name="destination"
            class="calc__input calc__input--text"
            type="text"
            list="cicles-list"
            placeholder="Digite a cidade ou local de destino"
            aria-required="true"
            autocomplete="off"
          />
        </div>

        <div class="calc__form-row">
          <label class="calc__label" for="distance">Distância (km)</label>
          <input
            id="distance"
            name="distance"
            class="calc__input calc__input--number"
            type="number"
            readonly
            aria-describedby="distance-help"
            min="0"
            step="0.1"
          />
          <p id="distance-help" class="calc__help">A distância será preenchida automaticamente</p>
        </div>

        <div class="calc__form-row calc__form-row--manual">
          <input
            id="manual-distance"
            name="manual-distance"
            class="calc__checkbox"
            type="checkbox"
            aria-controls="distance"
          />
          <label class="calc__label calc__label--checkbox" for="manual-distance">Inserir manualmente</label>
        </div>
      </fieldset>

      <fieldset class="calc__fieldset calc__fieldset--transport">
        <legend class="calc__legend">Modo de Transporte</legend>

        <div class="calc__transport-grid" role="radiogroup" aria-label="Modo de transporte">
          <label class="calc__transport-item" for="transport-bicycle">
            <input class="calc__transport-input" id="transport-bicycle" name="transport" type="radio" value="bicycle" />
            <span class="calc__transport-icon" aria-hidden="true">🚲</span>
            <span class="calc__transport-label">Bicicleta</span>
          </label>

          <label class="calc__transport-item" for="transport-car">
            <input class="calc__transport-input" id="transport-car" name="transport" type="radio" value="car" checked />
            <span class="calc__transport-icon" aria-hidden="true">🚗</span>
            <span class="calc__transport-label">Carro</span>
          </label>

          <label class="calc__transport-item" for="transport-bus">
            <input class="calc__transport-input" id="transport-bus" name="transport" type="radio" value="bus" />
            <span class="calc__transport-icon" aria-hidden="true">🚌</span>
            <span class="calc__transport-label">Ônibus</span>
          </label>

          <label class="calc__transport-item" for="transport-truck">
            <input class="calc__transport-input" id="transport-truck" name="transport" type="radio" value="truck" />
            <span class="calc__transport-icon" aria-hidden="true">🚚</span>
            <span class="calc__transport-label">Caminhão</span>
          </label>
        </div>
      </fieldset>

      <div class="calc__actions">
        <button class="calc__button" type="submit">Calcular Emissão</button>
      </div>
    </form>

    <section id="results" class="calc__results hidden" aria-hidden="true">
      <h2 class="calc__results-title">Resultados</h2>
      <div id="results-content" class="calc__results-content"></div>
    </section>

    <section id="comparison" class="calc__comparison hidden" aria-hidden="true">
      <h2 class="calc__comparison-title">Comparação</h2>
      <div id="comparison-content" class="calc__comparison-content"></div>
    </section>

    <section id="carbon-credits" class="calc__carbon-credits hidden" aria-hidden="true">
      <h2 class="calc__carbon-credits-title">Créditos de Carbono</h2>
      <div id="carbon-credits-content" class="calc__carbon-credits-content"></div>
    </section>
  </main>

  <footer class="calc__footer">
    <p class="calc__credits">Desenvolvido para a DIO | Projeto GitHub Copilot</p>
  </footer>

  <script src="js/routes-data.js"></script>
  <script src="js/config.js"></script>
  <script src="js/calculator.js"></script>
  <script src="js/ui.js"></script>
  // Ajustes visuais e acessibilidade para o grid de transporte
document.addEventListener('DOMContentLoaded', () => {
  const transportInputs = Array.from(document.querySelectorAll('.calc__transport-input'));

  if (!transportInputs.length) return;

  // Inicializa labels/estado com base no radio checked
  function updateSelected() {
    transportInputs.forEach(input => {
      const label = input.closest('.calc__transport-item');
      if (!label) return;
      if (input.checked) {
        label.classList.add('calc__transport-item--selected');
        label.setAttribute('aria-checked', 'true');
      } else {
        label.classList.remove('calc__transport-item--selected');
        label.setAttribute('aria-checked', 'false');
      }
    });
  }

  // Remove focus class from all labels
  function clearFocus() {
    transportInputs.forEach(i => {
      const lbl = i.closest('.calc__transport-item');
      if (lbl) lbl.classList.remove('calc__transport-item--focus');
    });
  }

  // Adiciona listeners
  transportInputs.forEach(input => {
    const label = input.closest('.calc__transport-item');
    if (!label) return;

    // Make the label behave like a radio for assistive tech
    label.setAttribute('role', 'radio');
    label.setAttribute('tabindex', '-1'); // input still receives focus
    label.setAttribute('aria-checked', input.checked ? 'true' : 'false');

    // quando o input muda, atualiza seleção
    input.addEventListener('change', () => {
      updateSelected();
    });

    // foco via teclado: marca visualmente o label
    input.addEventListener('focus', () => {
      clearFocus();
      label.classList.add('calc__transport-item--focus');
    });

    input.addEventListener('blur', () => {
      label.classList.remove('calc__transport-item--focus');
    });

    // permite seleção via clique no label (já padrão), mas adiciona foco ao input
    label.addEventListener('click', (e) => {
      // garantir que o input receba foco e seja checked
      if (!input.checked) {
        input.checked = true;
        input.dispatchEvent(new Event('change', { bubbles: true }));
      }
      input.focus();
    });

    // suporte de teclado extra: teclas arrow para navegar entre opções
    input.addEventListener('keydown', (e) => {
      const radios = transportInputs;
      const idx = radios.indexOf(input);
      if (e.key === 'ArrowRight' || e.key === 'ArrowDown') {
        e.preventDefault();
        const next = radios[(idx + 1) % radios.length];
        next.checked = true;
        next.focus();
        next.dispatchEvent(new Event('change', { bubbles: true }));
      } else if (e.key === 'ArrowLeft' || e.key === 'ArrowUp') {
        e.preventDefault();
        const prev = radios[(idx - 1 + radios.length) % radios.length];
        prev.checked = true;
        prev.focus();
        prev.dispatchEvent(new Event('change', { bubbles: true }));
      }
    });
  });

  // estado inicial
  updateSelected();
});
  <script src="js/app.js"></script>
</body>
</html>
