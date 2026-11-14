<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>FinTrack | Controle Financeiro Pessoal</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Inter', sans-serif; }
  </style>
</head>
<body class="bg-slate-900 text-slate-100 min-h-screen">

  <!-- Cabeçalho -->
  <header class="border-b border-slate-800 bg-slate-950/60 backdrop-blur-md sticky top-0 z-10">
    <div class="max-w-6xl mx-auto px-4 py-4 flex justify-between items-center">
      <div class="flex items-center gap-2">
        <div class="w-8 h-8 rounded-lg bg-emerald-500 flex items-center justify-center font-bold text-slate-950">F</div>
        <h1 class="text-xl font-bold tracking-tight">Fin<span class="text-emerald-400">Track</span></h1>
      </div>
      <button id="clear-data-btn" class="text-xs text-slate-400 hover:text-rose-400 transition">
        Limpar Dados
      </button>
    </div>
  </header>

  <main class="max-w-6xl mx-auto px-4 py-8 space-y-8">
    
    <!-- Cards de Resumo -->
    <section class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <div class="bg-slate-800/50 border border-slate-700/50 p-6 rounded-2xl">
        <p class="text-sm text-slate-400 font-medium">Entradas</p>
        <p id="total-income" class="text-2xl font-bold text-emerald-400 mt-2">R$ 0,00</p>
      </div>
      <div class="bg-slate-800/50 border border-slate-700/50 p-6 rounded-2xl">
        <p class="text-sm text-slate-400 font-medium">Saídas</p>
        <p id="total-expense" class="text-2xl font-bold text-rose-400 mt-2">R$ 0,00</p>
      </div>
      <div class="bg-slate-800 border border-slate-700 p-6 rounded-2xl shadow-lg">
        <p class="text-sm text-slate-400 font-medium">Saldo Atual</p>
        <p id="total-balance" class="text-2xl font-bold text-white mt-2">R$ 0,00</p>
      </div>
    </section>

    <!-- Área Principal: Formulário + Gráfico -->
    <section class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      
      <!-- Nova Transação -->
      <div class="bg-slate-800/50 border border-slate-700/50 p-6 rounded-2xl lg:col-span-1">
        <h2 class="text-lg font-semibold mb-4">Nova Transação</h2>
        <form id="transaction-form" class="space-y-4">
          <div>
            <label class="block text-xs text-slate-400 mb-1">Descrição</label>
            <input type="text" id="desc" required placeholder="Ex: Salário, Mercado, Aluguel"
              class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-emerald-500">
          </div>

          <div class="grid grid-cols-2 gap-3">
            <div>
              <label class="block text-xs text-slate-400 mb-1">Valor (R$)</label>
              <input type="number" id="amount" step="0.01" min="0.01" required placeholder="0,00"
                class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-emerald-500">
            </div>
            <div>
              <label class="block text-xs text-slate-400 mb-1">Tipo</label>
              <select id="type"
                class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-emerald-500">
                <option value="income">Entrada (+)</option>
                <option value="expense">Saída (-)</option>
              </select>
            </div>
          </div>

          <div>
            <label class="block text-xs text-slate-400 mb-1">Categoria</label>
            <select id="category"
              class="w-full bg-slate-900 border border-slate-700 rounded-lg px-3 py-2 text-sm focus:outline-none focus:border-emerald-500">
              <option value="Salário">Salário / Renda</option>
              <option value="Alimentação">Alimentação</option>
              <option value="Moradia">Moradia</option>
              <option value="Transporte">Transporte</option>
              <option value="Lazer">Lazer</option>
              <option value="Outros">Outros</option>
            </select>
          </div>

          <button type="submit"
            class="w-full bg-emerald-500 hover:bg-emerald-600 text-slate-950 font-semibold py-2.5 rounded-lg transition cursor-pointer">
            Adicionar Transação
          </button>
        </form>
      </div>

      <!-- Gráfico de Despesas por Categoria -->
      <div class="bg-slate-800/50 border border-slate-700/50 p-6 rounded-2xl lg:col-span-2 flex flex-col justify-between">
        <h2 class="text-lg font-semibold mb-4">Distribuição de Saídas por Categoria</h2>
        <div class="relative h-64 w-full flex items-center justify-center">
          <canvas id="expense-chart"></canvas>
          <p id="empty-chart-msg" class="text-sm text-slate-500 hidden">Adicione uma saída para visualizar o gráfico.</p>
        </div>
      </div>

    </section>

    <!-- Histórico de Transações -->
    <section class="bg-slate-800/50 border border-slate-700/50 p-6 rounded-2xl">
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 mb-6">
        <h2 class="text-lg font-semibold">Histórico de Movimentações</h2>
        <select id="filter-category"
          class="bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-sm focus:outline-none focus:border-emerald-500">
          <option value="ALL">Todas as Categorias</option>
          <option value="Salário">Salário / Renda</option>
          <option value="Alimentação">Alimentação</option>
          <option value="Moradia">Moradia</option>
          <option value="Transporte">Transporte</option>
          <option value="Lazer">Lazer</option>
          <option value="Outros">Outros</option>
        </select>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse">
          <thead>
            <tr class="border-b border-slate-700 text-xs text-slate-400 uppercase">
              <th class="py-3 px-4">Descrição</th>
              <th class="py-3 px-4">Categoria</th>
              <th class="py-3 px-4">Data</th>
              <th class="py-3 px-4 text-right">Valor</th>
              <th class="py-3 px-4 text-center">Ação</th>
            </tr>
          </thead>
          <tbody id="transaction-list" class="divide-y divide-slate-800 text-sm">
            <!-- Itens renderizados via JS -->
          </tbody>
        </table>
        <p id="empty-list-msg" class="text-center text-slate-500 py-8 text-sm">Nenhuma transação encontrada.</p>
      </div>
    </section>

  </main>

  <script src="script.js"></script>
</body>
</html>
