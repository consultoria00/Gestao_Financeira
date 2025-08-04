<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador de Custos de Empréstimo - PORTUS</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Corporate Calm -->
    <!-- Application Structure Plan: A dashboard-style SPA centered around an interactive loan simulator. This structure was chosen to transform the passive experience of reading a report into an active, exploratory one. The user can immediately see the financial impact of different loan scenarios, answering their primary question ("how much does this costs me?") upfront. Detailed textual information from the report is organized into accessible tabs below the simulator, reducing initial cognitive load while keeping all details available on demand. This serves both executives needing a quick overview and participants requiring in-depth information. The addition of a full amortization table provides a comprehensive cost overview. -->
    <!-- Visualization & Content Choices: The core of the application is the interactive simulator (Goal: Inform, Compare). Users manipulate sliders and dropdowns for loan amount, term, and age. The most complex data, the QQM/Taxa de Risco matrix (Goal: Organize, Inform), is handled by a JS lookup function, making it easy to understand. The output is shown as dynamic text and a Chart.js Donut Chart (Goal: Compare Proportions) visualizing the composition of origination fees. This is more effective than a static table. The new amortization table (Goal: Compare, Organize, Inform) provides a detailed breakdown of each monthly payment, enhancing transparency. Detailed text is placed in tabs (Goal: Organize) using JS to show/hide content, following the principle of progressive disclosure for a cleaner UI. The percentages for each fee are now explicitly displayed for enhanced user clarity. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8f9fa;
        }
        .tab-button.active {
            border-color: #2563eb;
            color: #2563eb;
            font-weight: 600;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 400px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 350px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 350px;
            }
        }
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 20px;
            height: 20px;
            background: #2563eb;
            cursor: pointer;
            border-radius: 50%;
        }
        input[type=range]::-moz-range-thumb {
            width: 20px;
            height: 20px;
            background: #2563eb;
            cursor: pointer;
            border-radius: 50%;
        }
        .table-container {
            overflow-x: auto;
            -webkit-overflow-scrolling: touch; /* smooth scrolling on iOS */
        }
        table {
            width: 100%;
            border-collapse: collapse;
        }
        th, td {
            padding: 8px 12px;
            border: 1px solid #e2e8f0;
            text-align: right;
            white-space: nowrap; /* Prevent wrapping in table cells */
        }
        th {
            background-color: #f1f5f9;
            font-weight: 600;
            text-align: center;
        }
        tbody tr:nth-child(even) {
            background-color: #f8fafc;
        }
    </style>
</head>
<body class="text-gray-800">

    <div class="container mx-auto p-4 md:p-8">
        
        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-bold text-gray-900">Estrutura de Custos de Empréstimos PORTUS</h1>
            <p class="mt-2 text-lg text-gray-600">Uma análise interativa dos encargos de empréstimo.</p>
        </header>

        <main>
            <section id="simulator" class="bg-white p-6 md:p-8 rounded-2xl shadow-lg mb-8">
                <h2 class="text-2xl font-bold mb-6 text-center">Simulador Interativo de Custos</h2>
                <p class="text-center text-gray-600 mb-8">Ajuste os valores abaixo para entender como os diferentes fatores impactam os custos do seu empréstimo. O simulador detalha os encargos cobrados no momento da concessão do crédito e a estimativa da primeira parcela mensal.</p>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-8 items-center">
                    
                    <div class="space-y-6">
                        <div>
                            <label for="loanAmount" class="block text-sm font-medium text-gray-700">Valor do Empréstimo: <span id="loanAmountLabel" class="font-bold text-blue-600">R$ 50.000</span></label>
                            <input type="range" id="loanAmount" min="1000" max="100000" step="1000" value="50000" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer">
                        </div>
                        
                        <div>
                            <label for="loanTerm" class="block text-sm font-medium text-gray-700">Prazo de Amortização</label>
                            <select id="loanTerm" class="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm rounded-md">
                                <option value="0">1 mês</option>
                                <option value="1">6 meses</option>
                                <option value="2">12 meses</option>
                                <option value="3">18 meses</option>
                                <option value="4">24 meses</option>
                                <option value="5">30 meses</option>
                                <option value="6">36 meses</option>
                                <option value="7">42 meses</option>
                                <option value="8">48 meses</option>
                                <option value="9">54 meses</option>
                                <option value="10">60 meses</option>
                            </select>
                        </div>

                        <div>
                            <label for="ageRange" class="block text-sm font-medium text-gray-700">Faixa Etária</label>
                            <select id="ageRange" class="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm rounded-md">
                                <option value="0">Entre 18 e 59 anos</option>
                                <option value="1">Entre 60 e 64 anos</option>
                                <option value="2">Entre 65 e 69 anos</option>
                                <option value="3">Entre 70 e 74 anos</option>
                                <option value="4">Entre 75 e 79 anos</option>
                                <option value="5">Acima de 79 anos</option>
                            </select>
                        </div>
                    </div>

                    <div class="flex flex-col items-center">
                        <div class="chart-container">
                            <canvas id="feesChart"></canvas>
                        </div>
                        <p class="text-sm text-gray-500 mt-2 text-center">Composição dos Encargos de Originação</p>
                    </div>
                </div>

                <div class="mt-10 border-t pt-8">
                    <h3 class="text-xl font-bold text-center mb-6">Resultados da Simulação</h3>
                    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 text-center mb-8">
                        <div class="bg-blue-50 p-4 rounded-lg">
                            <p class="text-sm text-blue-800">Taxa de Aprovação (TAC) <span id="tacPercent" class="font-bold text-blue-900 text-xs"></span></p>
                            <p id="tacValue" class="text-2xl font-bold text-blue-900">R$ 0,00</p>
                        </div>
                        <div class="bg-green-50 p-4 rounded-lg">
                            <p class="text-sm text-green-800">Taxa de Risco (QQM) <span id="qqmPercent" class="font-bold text-green-900 text-xs"></span></p>
                            <p id="qqmValue" class="text-2xl font-bold text-green-900">R$ 0,00</p>
                        </div>
                        <div class="bg-yellow-50 p-4 rounded-lg">
                            <p class="text-sm text-yellow-800">IOF <span id="iofPercent" class="font-bold text-yellow-900 text-xs"></span></p>
                            <p id="iofValue" class="text-2xl font-bold text-yellow-900">R$ 0,00</p>
                        </div>
                         <div class="bg-red-50 p-4 rounded-lg">
                            <p class="text-sm text-red-800">Total de Encargos Iniciais</p>
                            <p id="totalFees" class="text-2xl font-bold text-red-900">R$ 0,00</p>
                        </div>
                    </div>
                    <div class="mt-6 bg-gray-100 p-6 rounded-lg text-center">
                        <p class="text-lg text-gray-700">Valor Líquido a Receber</p>
                        <p id="netAmount" class="text-4xl font-extrabold text-gray-900">R$ 0,00</p>
                    </div>

                    <div class="mt-10 border-t pt-8">
                        <h3 class="text-xl font-bold text-center mb-6">Tabela de Amortização Mensal Estimada (Sistema SAC)</h3>
                        <p class="text-center text-gray-600 mb-6">Esta tabela detalha a composição de cada parcela mensal, incluindo juros, correção monetária, taxa administrativa, custo total, valor amortizado e saldo devedor. O valor do seguro mensal não está incluído devido à complexidade de sua determinação.</p>
                        <div class="table-container rounded-lg shadow-md">
                            <table class="min-w-full bg-white">
                                <thead>
                                    <tr>
                                        <th class="py-2 px-4 bg-gray-100 text-gray-700 text-sm font-semibold uppercase tracking-wider">Parcela</th>
                                        <th class="py-2 px-4 bg-gray-100 text-gray-700 text-sm font-semibold uppercase tracking-wider">Valor da Parcela</th>
                                        <th class="py-2 px-4 bg-gray-100 text-gray-700 text-sm font-semibold uppercase tracking-wider">Juros</th>
                                        <th class="py-2 px-4 bg-gray-100 text-gray-700 text-sm font-semibold uppercase tracking-wider">Correção Monetária</th>
                                        <th class="py-2 px-4 bg-gray-100 text-gray-700 text-sm font-semibold uppercase tracking-wider">Taxa Administrativa</th>
                                        <th class="py-2 px-4 bg-gray-100 text-gray-700 text-sm font-semibold uppercase tracking-wider">Custo Total</th>
                                        <th class="py-2 px-4 bg-gray-100 text-gray-700 text-sm font-semibold uppercase tracking-wider">Valor Amortizado</th>
                                        <th class="py-2 px-4 bg-gray-100 text-gray-700 text-sm font-semibold uppercase tracking-wider">Saldo Devedor</th>
                                    </tr>
                                </thead>
                                <tbody id="amortizationTableBody">
                                    <!-- Rows will be dynamically inserted here -->
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </section>

            <section id="details" class="bg-white p-6 md:p-8 rounded-2xl shadow-lg">
                <h2 class="text-2xl font-bold mb-6 text-center">Detalhamento dos Encargos</h2>
                <p class="text-center text-gray-600 mb-8">Selecione uma categoria para ler a explicação detalhada de cada componente do custo do seu empréstimo.</p>
                <div class="border-b border-gray-200 mb-6">
                    <nav class="-mb-px flex justify-center space-x-4 md:space-x-8" aria-label="Tabs">
                        <button data-tab="origination" class="tab-button active whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                            Encargos de Originação
                        </button>
                        <button data-tab="maintenance" class="tab-button whitespace-nowrap py-4 px-1 border-b-2 border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300 font-medium text-sm">
                            Encargos de Manutenção
                        </button>
                        <button data-tab="final" class="tab-button whitespace-nowrap py-4 px-1 border-b-2 border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300 font-medium text-sm">
                            Considerações Finais
                        </button>
                    </nav>
                </div>

                <div id="tab-content" class="prose max-w-none">
                    <div id="origination-content" class="tab-pane">
                        <h3 class="font-bold text-lg mb-2">Taxa de Aprovação de Crédito (TAC)</h3>
                        <p class="mb-4">Este encargo financeiro é cobrado no momento da concessão, repactuação ou novação do crédito. Sua finalidade é cobrir os custos administrativos e operacionais associados à análise, aprovação e formalização do Contrato de Empréstimo. O valor corresponde a 0,5% sobre o montante total do contrato. É uma cobrança única, realizada no início da operação.</p>
                        
                        <h3 class="font-bold text-lg mb-2">Taxa de Risco (QQM - Quota de Quitação por Morte)</h3>
                        <p class="mb-4">A Taxa de Risco é um encargo destinado à constituição de um Fundo de Quitação por Morte (FQM), que garante a liquidação integral do saldo devedor do empréstimo em caso de falecimento do mutuário. Este encargo é cobrado em parcela única no momento da concessão do crédito. As alíquotas são variáveis, dependendo da faixa etária do mutuário e do prazo de amortização do empréstimo.</p>

                        <h3 class="font-bold text-lg mb-2">IOF (Imposto sobre Operações Financeiras)</h3>
                        <p>O IOF é um tributo federal incidente sobre operações de crédito. No contexto dos empréstimos da PORTUS, é aplicado sobre o valor total contratado, conforme a legislação vigente. Sua alíquota e metodologia de cálculo são definidas pela Receita Federal do Brasil.</p>
                    </div>
                    <div id="maintenance-content" class="tab-pane hidden">
                        <h3 class="font-bold text-lg mb-2">Taxa de Juros Real</h3>
                        <p class="mb-4">Representa o principal componente de remuneração do capital emprestado. A taxa nominal é de 5,21% ao ano, aplicada mensalmente sobre o saldo devedor. A definição desta taxa considera a Meta Atuarial dos planos de benefícios (INPC + 4,71%), acrescida de 0,48% para cobrir eventuais inadimplências e flutuações de mercado, configurando um custo fixo para o mutuário durante a amortização.</p>

                        <h3 class="font-bold text-lg mb-2">Atualização Monetária</h3>
                        <p class="mb-4">Tem como objetivo preservar o poder de compra do capital emprestado, corrigindo o saldo devedor pela inflação. A PORTUS utiliza o Índice Nacional de Preços ao Consumidor (INPC) como indexador, com uma defasagem de dois meses. A adoção do INPC visa assegurar consistência com a meta atuarial dos planos de benefícios.</p>

                        <h3 class="font-bold text-lg mb-2">Taxa de Administração</h3>
                        <p>Encargo mensal para cobrir os custos operacionais e de gestão do contrato. A taxa é de 0,41% ao mês (equivalente a 5% ao ano) e incide sobre o saldo devedor. Sua correta calibração é fundamental para a sustentabilidade da operação de empréstimos.</p>
                    </div>
                    <div id="final-content" class="tab-pane hidden">
                        <p>É de suma importância que os mutuários da PORTUS compreendam integralmente a estrutura da taxa de juros e os encargos incidentes. A transparência na apresentação dessas informações é crucial para que o mutuário possa tomar decisões financeiras conscientes. As taxas e encargos aqui apresentados podem sofrer alterações, em conformidade com o Contrato de Abertura de Crédito e o Regulamento de Empréstimos. Eventuais modificações nas condições serão realizadas a critério da Diretoria Executiva da PORTUS ou em função de alterações na legislação aplicável. Recomenda-se que os mutuários consultem sempre os documentos oficiais da PORTUS e, em caso de dúvidas, busquem esclarecimentos junto aos canais de atendimento da instituição.</p>
                    </div>
                </div>
            </section>
        </main>

        <footer class="text-center mt-12 py-4 border-t">
            <p class="text-sm text-gray-500">&copy; 2025 PORTUS. Todos os direitos reservados.</p>
        </footer>

    </div>

    <script>
        const loanAmountInput = document.getElementById('loanAmount');
        const loanAmountLabel = document.getElementById('loanAmountLabel');
        const loanTermSelect = document.getElementById('loanTerm');
        const ageRangeSelect = document.getElementById('ageRange');

        const tacValueEl = document.getElementById('tacValue');
        const qqmValueEl = document.getElementById('qqmValue');
        const iofValueEl = document.getElementById('iofValue');
        const totalFeesEl = document.getElementById('totalFees');
        const netAmountEl = document.getElementById('netAmount');

        const tacPercentEl = document.getElementById('tacPercent');
        const qqmPercentEl = document.getElementById('qqmPercent');
        const iofPercentEl = document.getElementById('iofPercent');
        // Removidas referências a interestAdminPercentEl e correctionPercentEl pois não existem mais no HTML
        // const interestAdminPercentEl = document.getElementById('interestAdminPercent');
        // const correctionPercentEl = document.getElementById('correctionPercent');


        const amortizationTableBody = document.getElementById('amortizationTableBody');

        const tabButtons = document.querySelectorAll('.tab-button');
        const tabPanes = document.querySelectorAll('.tab-pane');

        const formatCurrency = (value) => {
            return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value);
        };

        const formatPercent = (value) => {
            return new Intl.NumberFormat('pt-BR', { style: 'percent', minimumFractionDigits: 2, maximumFractionDigits: 4 }).format(value);
        };

        const qqmRates = [
            [0.0006909, 0.002947, 0.004919, 0.008315, 0.013507, 0.028696],
            [0.001630, 0.006978, 0.011681, 0.019614, 0.031662, null],
            [0.002204, 0.009466, 0.015874, 0.026524, null, null],
            [0.003176, 0.013719, 0.023043, null, null, null],
            [0.004468, 0.019470, null, null, null, null],
            [0.005797, null, null, null, null, null]
        ];

        // Mapeamento do índice do prazo para o número real de meses
        const actualMonths = [1, 6, 12, 18, 24, 30, 36, 42, 48, 54, 60]; 

        const tacRate = 0.005; // 0.5%
        const iofRate = 0.0038; // 0.38%
        const monthlyInterestRate = 0.0042; // 0.42% a.m. - Taxa de Juros Real
        const monthlyAdminRate = 0.0041; // 0.41% a.m. - Taxa de Administração
        const monthlyINPCRate = 0.0041; // 0.41% a.m. - INPC fixo conforme solicitação

        let feesChart;

        function initializeChart() {
            const ctx = document.getElementById('feesChart').getContext('2d');
            feesChart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: ['TAC', 'QQM', 'IOF'],
                    datasets: [{
                        data: [0, 0, 0],
                        backgroundColor: ['#3b82f6', '#10b981', '#f59e0b'],
                        borderColor: '#ffffff',
                        borderWidth: 2,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    cutout: '60%',
                    plugins: {
                        legend: {
                            position: 'bottom',
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed !== null) {
                                        label += formatCurrency(context.parsed);
                                    }
                                    return label;
                                }
                            }
                        }
                    }
                }
            });
        }

        function calculatePMT(principal, monthlyRate, numPayments) {
            if (monthlyRate === 0) return principal / numPayments;
            return principal * (monthlyRate * Math.pow(1 + monthlyRate, numPayments)) / (Math.pow(1 + monthlyRate, numPayments) - 1);
        }

        // Função para mapear o número de meses para o índice correto da matriz qqmRates
        function getQqmTermIndex(numMonths) {
            if (numMonths >= 1 && numMonths <= 12) return 0;
            if (numMonths >= 13 && numMonths <= 18) return 1;
            if (numMonths >= 19 && numMonths <= 24) return 2;
            if (numMonths >= 25 && numMonths <= 36) return 3;
            if (numMonths >= 37 && numMonths <= 48) return 4;
            if (numMonths >= 49 && numMonths <= 60) return 5;
            return -1; // Retorna -1 para casos não mapeados, embora as opções de dropdown devam cobrir todos
        }

        function updateSimulation() {
            const amount = parseFloat(loanAmountInput.value);
            const termIndex = parseInt(loanTermSelect.value);
            const ageIndex = parseInt(ageRangeSelect.value);
            const numMonths = actualMonths[termIndex];

            loanAmountLabel.textContent = formatCurrency(amount);

            // Cálculos dos Encargos de Originação
            const tacValue = amount * tacRate;
            const iofValue = amount * iofRate;

            // Calcular o valor base para QQM após deduzir TAC e IOF
            const baseAmountForQqm = amount - tacValue - iofValue;

            const qqmTermArrayIndex = getQqmTermIndex(numMonths); 
            
            let qqmValue = 0;
            let currentQqmPercent = 0;

            if (qqmTermArrayIndex !== -1 && qqmRates[qqmTermArrayIndex] && qqmRates[qqmTermArrayIndex][ageIndex] !== null) {
                currentQqmPercent = qqmRates[qqmTermArrayIndex][ageIndex];
                qqmValue = baseAmountForQqm * currentQqmPercent;
            } else {
                qqmValue = 0;
                currentQqmPercent = 0;
            }
            
            const totalFees = tacValue + qqmValue + iofValue;
            const netAmount = amount - totalFees;

            tacValueEl.textContent = formatCurrency(tacValue);
            qqmValueEl.textContent = formatCurrency(qqmValue);
            iofValueEl.textContent = formatCurrency(iofValue);
            totalFeesEl.textContent = formatCurrency(totalFees);
            netAmountEl.textContent = formatCurrency(netAmount);

            // Exibir percentuais
            tacPercentEl.textContent = `(${formatPercent(tacRate)})`;
            qqmPercentEl.textContent = `(${formatPercent(currentQqmPercent)})`;
            iofPercentEl.textContent = `(${formatPercent(iofRate)})`;


            updateChartData(tacValue, qqmValue, iofValue);

            // Cálculos da Tabela de Amortização (Sistema SAC)
            amortizationTableBody.innerHTML = ''; // Limpa a tabela antes de preencher
            
            let currentSaldoDevedor = netAmount;
            const constantAmortization = netAmount / numMonths; // Amortização constante do principal

            for (let i = 1; i <= numMonths; i++) {
                // 1. Calcular Correção Monetária sobre o saldo devedor inicial do mês
                let correcaoMes = currentSaldoDevedor * monthlyINPCRate;

                // 2. Saldo devedor atualizado pela correção monetária para cálculo de Juros
                let saldoDevedorCorrigidoParaJuros = currentSaldoDevedor + correcaoMes;

                // 3. Calcular Juros sobre o saldo devedor corrigido
                let jurosMes = saldoDevedorCorrigidoParaJuros * monthlyInterestRate;

                // 4. Calcular Taxa Administrativa sobre o saldo devedor original do mês (currentSaldoDevedor)
                let adminMes = currentSaldoDevedor * monthlyAdminRate; // Ajuste aqui!

                // 5. Custo Total do mês (Juros + Correção Monetária + Taxa Administrativa)
                let custoTotalMes = jurosMes + correcaoMes + adminMes;
                
                let valorAmortizado = constantAmortization;
                let valorParcela = valorAmortizado + custoTotalMes;

                // Ajuste para a última parcela para garantir que o saldo devedor seja zero
                if (i === numMonths) {
                    valorAmortizado = currentSaldoDevedor; // Amortiza o restante do principal
                    // Recalcula os custos para a última parcela com o saldo final
                    jurosMes = (currentSaldoDevedor + (currentSaldoDevedor * monthlyINPCRate)) * monthlyInterestRate; // Juros sobre saldo corrigido final
                    correcaoMes = currentSaldoDevedor * monthlyINPCRate;
                    adminMes = currentSaldoDevedor * monthlyAdminRate; // Admin sobre saldo original final
                    custoTotalMes = jurosMes + correcaoMes + adminMes;
                    valorParcela = valorAmortizado + custoTotalMes; // Ajusta a parcela final
                }

                currentSaldoDevedor -= valorAmortizado;
                // Garante que o saldo não seja negativo devido a imprecisões de ponto flutuante
                if (currentSaldoDevedor < 0.01 && i === numMonths) { // Pequena tolerância para zero
                    currentSaldoDevedor = 0;
                }

                const row = amortizationTableBody.insertRow();
                row.insertCell().textContent = i;
                row.insertCell().textContent = formatCurrency(valorParcela);
                row.insertCell().textContent = formatCurrency(jurosMes);
                row.insertCell().textContent = formatCurrency(correcaoMes);
                row.insertCell().textContent = formatCurrency(adminMes);
                row.insertCell().textContent = formatCurrency(custoTotalMes);
                row.insertCell().textContent = formatCurrency(valorAmortizado);
                row.insertCell().textContent = formatCurrency(currentSaldoDevedor);
            }
        }

        function updateChartData(tac, qqm, iof) {
            feesChart.data.datasets[0].data = [tac, qqm, iof];
            feesChart.update();
        }
        
        function handleTabClick(event) {
            const clickedTab = event.currentTarget;
            const tabName = clickedTab.dataset.tab;

            tabButtons.forEach(button => {
                button.classList.remove('active');
            });
            clickedTab.classList.add('active');

            tabPanes.forEach(pane => {
                if (pane.id === `${tabName}-content`) {
                    pane.classList.remove('hidden');
                } else {
                    pane.classList.add('hidden');
                }
            });
        }

        loanAmountInput.addEventListener('input', updateSimulation);
        loanTermSelect.addEventListener('change', updateSimulation);
        ageRangeSelect.addEventListener('change', updateSimulation);
        
        tabButtons.forEach(button => button.addEventListener('click', handleTabClick));

        document.addEventListener('DOMContentLoaded', () => {
            initializeChart();
            updateSimulation();
        });

    </script>
</body>
</html>
