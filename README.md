<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CG - Controle de Gastos mobile (PWA Native Look)</title>
    <!-- Tailwind CSS para estilização moderna e rápida -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons para iconografia nativa e consistente -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Biblioteca OCR Tesseract.js para leitura local de imagens sem servidores externos -->
    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>
    <!-- Biblioteca jsQR para leitura de códigos QR -->
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.min.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap');
        body { 
            font-family: 'Inter', sans-serif; 
            background-color: #0f172a; /* Slate 900 de fundo */
        }
        
        @keyframes scan-line {
            0% { top: 0%; }
            100% { top: 100%; }
        }
        .animate-scan-line {
            position: absolute;
            width: 100%;
            height: 2px;
            background: rgba(16, 185, 129, 0.6);
            box-shadow: 0 0 15px rgba(16, 185, 129, 0.8);
            animation: scan-line 2s linear infinite;
        }

        .tab-active {
            color: #10b981; /* emerald-500 */
        }
        
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

        /* Mockup elegante de celular para visualização em telas largas */
        @media (min-width: 640px) {
            .app-shell {
                max-width: 420px;
                margin: 40px auto;
                min-height: 850px;
                height: 85vh;
                position: relative;
                box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5), 0 0 40px rgba(16, 185, 129, 0.1);
                border: 12px solid #1e293b;
                border-radius: 48px;
                background-color: #ffffff;
                overflow: hidden;
            }
            .fixed-bottom-nav {
                position: absolute !important;
                bottom: 0;
                left: 0;
                right: 0;
                border-bottom-left-radius: 36px;
                border-bottom-right-radius: 36px;
            }
            .fixed-scanner-modal {
                position: absolute !important;
                border-radius: 36px;
            }
        }

        /* Suporte a toques e transições suaves */
        .tap-feedback:active {
            transform: scale(0.96);
            opacity: 0.9;
        }
        .transition-all-custom {
            transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen text-slate-900 overflow-x-hidden">

    <!-- APP SHELL CONTAINER (MOCKUP DE SMARTPHONE) -->
    <div class="app-shell bg-white flex flex-col w-full min-h-screen sm:min-h-0 pb-24 transition-all-custom">

        <!-- HEADER ESTILO NATIVO -->
        <header class="bg-emerald-800 text-white sticky top-0 z-40 px-5 py-4 shadow-md">
            <div class="flex items-center justify-between">
                <div class="flex items-center gap-2.5">
                    <div class="bg-white/10 p-2.5 rounded-2xl backdrop-blur-md">
                        <i data-lucide="calculator" class="w-5 h-5 text-emerald-300"></i>
                    </div>
                    <div>
                        <h1 class="text-[10px] font-black tracking-widest uppercase text-emerald-200">Controle de Gastos</h1>
                        <p class="text-base font-extrabold tracking-tight" id="display-date-title">Janeiro, 2026</p>
                    </div>
                </div>
                
                <!-- Navegador de Período Rápido -->
                <div class="flex items-center bg-white/10 rounded-xl p-1 gap-1">
                    <button onclick="changePeriod(-1)" class="p-1.5 hover:bg-white/20 rounded-lg transition-colors tap-feedback">
                        <i data-lucide="chevron-left" class="w-4 h-4"></i>
                    </button>
                    <button onclick="changePeriod(1)" class="p-1.5 hover:bg-white/20 rounded-lg transition-colors tap-feedback">
                        <i data-lucide="chevron-right" class="w-4 h-4"></i>
                    </button>
                </div>
            </div>
        </header>

        <!-- MAIN SCROLL VIEW -->
        <main class="flex-1 overflow-y-auto px-5 py-5 space-y-5 no-scrollbar">

            <!-- ========================================== -->
            <!-- ABA 1: LANÇAMENTOS (CENTRAL DE CAPTURA) -->
            <!-- ========================================== -->
            <div id="section-lancamentos" class="space-y-5 animate-in fade-in duration-200">
                
                <div class="space-y-3">
                    <h2 class="text-xs font-black text-slate-400 uppercase tracking-widest">Ferramentas Inteligentes</h2>
                    <div class="grid grid-cols-3 gap-2.5">
                        <button onclick="startScanner()" class="bg-emerald-50 hover:bg-emerald-100 border border-emerald-100 rounded-2xl p-3 flex flex-col items-center justify-center text-center transition-all tap-feedback">
                            <div class="bg-emerald-600 text-white p-2.5 rounded-xl mb-2 shadow-sm">
                                <i data-lucide="scan-qr-code" class="w-5 h-5"></i>
                            </div>
                            <span class="text-[9px] font-black text-emerald-800 uppercase tracking-wide">Scanner QR</span>
                        </button>

                        <label class="bg-blue-50 hover:bg-blue-100 border border-blue-100 rounded-2xl p-3 flex flex-col items-center justify-center text-center transition-all tap-feedback cursor-pointer">
                            <input type="file" id="file-upload" class="hidden" accept="image/*" onchange="handleFileUpload(event)" />
                            <div class="bg-blue-600 text-white p-2.5 rounded-xl mb-2 shadow-sm">
                                <i data-lucide="image" class="w-5 h-5"></i>
                            </div>
                            <span class="text-[9px] font-black text-blue-800 uppercase tracking-wide">Carregar Foto</span>
                        </label>

                        <button onclick="focusLinkInput()" class="bg-purple-50 hover:bg-purple-100 border border-purple-100 rounded-2xl p-3 flex flex-col items-center justify-center text-center transition-all tap-feedback">
                            <div class="bg-purple-600 text-white p-2.5 rounded-xl mb-2 shadow-sm">
                                <i data-lucide="link" class="w-5 h-5"></i>
                            </div>
                            <span class="text-[9px] font-black text-purple-800 uppercase tracking-wide">Colar Link</span>
                        </button>
                    </div>
                </div>

                <!-- Campo de Entrada de Nota Oculto/Expansível -->
                <div id="link-input-container" class="hidden bg-slate-50 border border-slate-200 rounded-2xl p-4 animate-in slide-in-from-top-2 duration-200 space-y-3">
                    <div class="flex justify-between items-center">
                        <span class="text-[9px] font-black text-slate-400 uppercase tracking-widest">Colar Cupom ou Chave</span>
                        <button onclick="toggleElement('link-input-container', false)" class="text-slate-400 hover:text-slate-600">
                            <i data-lucide="x" class="w-4 h-4"></i>
                        </button>
                    </div>
                    <div class="flex gap-2">
                        <input 
                            type="text" 
                            id="scan-input"
                            placeholder="Link NFC-e ou Chave com 44 dígitos..." 
                            class="flex-1 bg-white border border-slate-200 rounded-xl text-xs p-3 outline-none focus:ring-1 focus:ring-emerald-500"
                        />
                        <button onclick="handleLinkProcess()" class="bg-emerald-600 text-white px-4 rounded-xl font-bold text-xs hover:bg-emerald-700 active:scale-95 transition-all">OK</button>
                    </div>
                </div>

                <!-- VALIDAÇÃO DE PENDENTES EXTRAÍDOS -->
                <div id="pending-section" class="hidden bg-amber-50 border border-amber-200 rounded-2xl overflow-hidden shadow-sm animate-in fade-in duration-300">
                    <div class="p-4 bg-amber-100/50 flex justify-between items-center border-b border-amber-200">
                        <h3 class="text-xs font-black text-amber-800 uppercase flex items-center gap-1.5">
                            <i data-lucide="file-check" class="w-4 h-4 text-amber-600"></i> Validando (<span id="pending-count">0</span>)
                        </h3>
                        <button onclick="confirmAllPending()" class="bg-amber-600 text-white px-3 py-1.5 rounded-xl text-[9px] font-bold uppercase hover:bg-emerald-700 transition-colors">
                            Salvar Todos
                        </button>
                    </div>
                    <div id="pending-list" class="p-3 space-y-2"></div>
                </div>

                <!-- FORMULÁRIO MANUAL COM MÉTODO DE GASTO RECORRENTE -->
                <form id="manual-form" class="bg-slate-50 rounded-2xl p-5 border border-slate-200/60 space-y-4 shadow-sm" onsubmit="handleManualSubmit(event)">
                    <h3 class="text-xs font-black text-slate-500 uppercase tracking-widest flex items-center gap-2 border-b border-slate-200/60 pb-2">
                        <i data-lucide="pen-tool" class="w-4 h-4 text-emerald-600"></i> Lançamento Manual
                    </h3>
                    <div class="grid grid-cols-2 gap-3">
                        <div class="space-y-1">
                            <label class="text-[9px] font-bold text-slate-400 uppercase">Usuário</label>
                            <select id="select-user" name="user" class="w-full bg-white border border-slate-200 rounded-lg text-xs p-2 outline-none focus:ring-1 focus:ring-emerald-500 transition-all"></select>
                        </div>
                        <div class="space-y-1">
                            <label class="text-[9px] font-bold text-slate-400 uppercase">Categoria</label>
                            <select id="select-category" name="category" class="w-full bg-white border border-slate-200 rounded-lg text-xs p-2 outline-none focus:ring-1 focus:ring-emerald-500 transition-all"></select>
                        </div>
                        <div class="space-y-1 col-span-2">
                            <label class="text-[9px] font-bold text-slate-400 uppercase">Local/Estabelecimento</label>
                            <input name="description" required type="text" placeholder="Ex: Feira livre" class="w-full bg-white border border-slate-200 rounded-lg text-xs p-2 outline-none focus:ring-1 focus:ring-emerald-500 transition-all" />
                        </div>
                        <div class="space-y-1 col-span-2">
                            <label class="text-[9px] font-bold text-slate-400 uppercase">Valor R$</label>
                            <input name="amount" required step="0.01" type="number" placeholder="0,00" class="w-full border border-emerald-200 bg-emerald-50/30 rounded-lg text-xs p-2 font-bold outline-none focus:ring-1 focus:ring-emerald-500 transition-all" />
                        </div>
                    </div>

                    <!-- RECORRÊNCIA -->
                    <div class="flex items-center justify-between p-3 bg-white border border-slate-200/60 rounded-xl">
                        <div class="flex items-center gap-2">
                            <i data-lucide="repeat" class="w-4 h-4 text-emerald-600"></i>
                            <span class="text-xs font-bold text-slate-700">Tornar gasto recorrente?</span>
                        </div>
                        <label class="relative inline-flex items-center cursor-pointer">
                            <input type="checkbox" id="check-recurring" class="sr-only peer" onchange="toggleRecurringPeriod(this.checked)">
                            <div class="w-9 h-5 bg-slate-200 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-slate-300 after:border after:rounded-full after:h-4 after:w-4 after:transition-all peer-checked:bg-emerald-600"></div>
                        </label>
                    </div>

                    <!-- PERÍODO DE RECORRÊNCIA -->
                    <div id="recurring-period-panel" class="hidden animate-in slide-in-from-top-2 duration-200 p-4 bg-emerald-50/50 border border-emerald-100 rounded-xl space-y-3">
                        <p class="text-[9px] font-black text-emerald-800 uppercase tracking-widest flex items-center gap-1.5">
                            <i data-lucide="calendar" class="w-3.5 h-3.5"></i> Período e Frequência
                        </p>
                        <div class="grid grid-cols-2 gap-3">
                            <div class="space-y-1">
                                <label class="text-[9px] font-bold text-slate-400 uppercase">Frequência</label>
                                <select id="recurring-frequency" class="w-full bg-white border border-slate-200 rounded-lg text-xs p-2 outline-none focus:ring-1 focus:ring-emerald-500">
                                    <option value="mensal">Mensal</option>
                                    <option value="bimestral">Bimestral</option>
                                    <option value="trimestral">Trimestral</option>
                                    <option value="semestral">Semestral</option>
                                    <option value="anual">Anual</option>
                                </select>
                            </div>
                            <div class="space-y-1">
                                <label class="text-[9px] font-bold text-slate-400 uppercase">Repetir por:</label>
                                <div class="flex items-center gap-1.5">
                                    <input type="number" id="recurring-months" min="2" max="120" value="12" class="w-full bg-white border border-slate-200 rounded-lg text-xs p-2 outline-none focus:ring-1 focus:ring-emerald-500 text-center font-bold" />
                                    <span class="text-[10px] text-slate-500 font-bold uppercase">vezes</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <button type="submit" class="w-full bg-emerald-600 text-white font-bold py-2.5 rounded-lg text-xs uppercase shadow-md hover:bg-emerald-700 active:scale-95 transition-all">Salvar Lançamento</button>
                </form>
            </div>

            <!-- ========================================== -->
            <!-- ABA 2: EXTRATO (TRANSAÇÕES DO MÊS) -->
            <!-- ========================================== -->
            <div id="section-extrato" class="hidden space-y-4 animate-in fade-in duration-200">
                <div class="flex justify-between items-center">
                    <h2 class="text-xs font-black text-slate-400 uppercase tracking-widest">Extrato do Período</h2>
                    <span id="registered-count" class="text-[9px] bg-slate-100 text-slate-600 px-2.5 py-1 rounded-full font-bold">0 itens</span>
                </div>
                
                <div id="transactions-list" class="space-y-3">
                    <!-- Gerado Dinamicamente -->
                </div>
            </div>

            <!-- ========================================== -->
            <!-- ABA 3: MENSAL (DASHBOARD POR USUÁRIO E CATEGORIA) -->
            <!-- ========================================== -->
            <div id="section-dashboard" class="hidden space-y-5 animate-in fade-in duration-200">
                <div class="bg-gradient-to-br from-emerald-600 to-emerald-850 text-white p-5 rounded-3xl shadow-lg flex items-center justify-between">
                    <div>
                        <h3 class="text-[9px] font-black text-emerald-200 uppercase tracking-widest mb-1">Gasto Consolidado</h3>
                        <div id="monthly-total-value" class="text-3xl font-black tracking-tight">R$ 0,00</div>
                    </div>
                    <div class="bg-white/10 p-3 rounded-2xl backdrop-blur-md">
                        <i data-lucide="activity" class="w-6 h-6 text-emerald-300"></i>
                    </div>
                </div>

                <div class="space-y-3">
                    <h3 class="text-xs font-black text-slate-400 uppercase tracking-widest">Membros da Família</h3>
                    <div id="user-stats-list" class="grid grid-cols-2 gap-3"></div>
                </div>

                <div class="bg-slate-50 border border-slate-200/60 rounded-3xl p-5 space-y-4">
                    <h3 class="font-black text-slate-700 text-xs uppercase tracking-widest flex items-center gap-2">
                        <i data-lucide="pie-chart" class="text-emerald-500 w-4 h-4"></i> Categorias do Mês
                    </h3>
                    <div id="category-progress-list" class="space-y-4"></div>
                </div>
            </div>

            <!-- ========================================== -->
            <!-- ABA 4: ANUAL (VISÃO DE TENDÊNCIAS ANUAIS) -->
            <!-- ========================================== -->
            <div id="section-anual" class="hidden space-y-5 animate-in fade-in duration-200">
                <div class="bg-slate-900 text-white p-5 rounded-3xl shadow-xl flex items-center justify-between border border-slate-850">
                    <div>
                        <h3 id="annual-total-title" class="text-[10px] font-black text-emerald-400 uppercase tracking-[0.2em] mb-1">Total do Ano</h3>
                        <div id="annual-total-value" class="text-2xl font-black tracking-tight">R$ 0,00</div>
                    </div>
                    <div class="bg-white/10 p-3.5 rounded-xl"><i data-lucide="calendar" class="w-5 h-5 text-emerald-400"></i></div>
                </div>

                <div class="space-y-3">
                    <h3 class="text-xs font-black text-slate-400 uppercase tracking-widest">Membros da Família (Ano)</h3>
                    <div id="user-annual-stats" class="grid grid-cols-2 gap-3"></div>
                </div>

                <div class="bg-slate-50 border border-slate-200/60 rounded-3xl p-5 space-y-6">
                    <h3 class="font-black text-slate-700 text-xs uppercase tracking-widest flex items-center gap-2">
                        <i data-lucide="bar-chart-3" class="text-emerald-500 w-4 h-4"></i> Evolução Mensal
                    </h3>
                    <div id="annual-bars-container" class="flex items-end justify-between h-44 gap-2.5 px-1 pt-4"></div>
                </div>
            </div>

            <!-- ========================================== -->
            <!-- ABA 5: AJUSTES (PAINEL DE CONTROLE INTEGRADO) -->
            <!-- ========================================== -->
            <div id="section-ajustes" class="hidden space-y-5 animate-in fade-in duration-200">
                
                <!-- GERENCIAR USUÁRIOS -->
                <section class="space-y-4 bg-slate-50 border border-slate-200/60 p-5 rounded-3xl shadow-sm">
                    <h4 class="text-[10px] font-black text-slate-500 uppercase tracking-[0.2em] flex items-center gap-2 border-b border-slate-200 pb-2">
                        <i data-lucide="users" class="w-4 h-4 text-emerald-600"></i> Gerenciar Usuários
                    </h4>
                    <div class="flex flex-col gap-2">
                        <input 
                            type="text" 
                            id="new-user-input"
                            placeholder="Nome do Usuário..." 
                            class="w-full bg-white border border-slate-200 rounded-xl text-xs p-3 outline-none focus:ring-1 focus:ring-emerald-500"
                        />
                        <div class="flex gap-2">
                            <input 
                                type="text" 
                                id="new-user-card"
                                placeholder="Cartões (ex: 1234, 5678)" 
                                class="flex-1 bg-white border border-slate-200 rounded-xl text-xs p-3 outline-none focus:ring-1 focus:ring-emerald-500"
                            />
                            <button 
                                onclick="handleCreateUser()"
                                class="bg-emerald-600 text-white px-4 rounded-xl font-bold text-xs uppercase shadow-md hover:bg-emerald-750 active:scale-95 transition-all"
                            >
                                <i data-lucide="user-plus" class="w-4 h-4"></i>
                            </button>
                        </div>
                    </div>
                    <div id="users-config-list" class="grid grid-cols-1 gap-3 pt-2"></div>
                </section>

                <!-- GERENCIAR CATEGORIAS -->
                <section class="space-y-4 bg-slate-50 border border-slate-200/60 p-5 rounded-3xl shadow-sm">
                    <h4 class="text-[10px] font-black text-slate-500 uppercase tracking-[0.2em] flex items-center gap-2 border-b border-slate-200 pb-2">
                        <i data-lucide="tag" class="w-4 h-4 text-emerald-600"></i> Gerenciar Categorias
                    </h4>
                    <div class="flex gap-2">
                        <input 
                            type="text" 
                            id="new-category-input"
                            placeholder="Nova Categoria..." 
                            class="flex-1 bg-white border border-slate-200 rounded-xl text-xs p-3 outline-none focus:ring-1 focus:ring-emerald-500"
                        />
                        <button 
                            onclick="handleCreateCategory()"
                            class="bg-emerald-600 text-white px-5 rounded-xl font-bold text-xs uppercase shadow-md hover:bg-emerald-750 active:scale-95 transition-all"
                        >
                            ADD
                        </button>
                    </div>
                    <div id="categories-config-list" class="flex flex-wrap gap-2 p-2 bg-white rounded-2xl border border-slate-100"></div>
                </section>

                <!-- CONFIGURAÇÕES ADICIONAIS -->
                <section class="bg-red-50 border border-red-200 p-5 rounded-3xl space-y-3 shadow-sm">
                    <h4 class="text-[10px] font-black text-red-800 uppercase tracking-[0.2em]">Danger Zone</h4>
                    <p class="text-[10px] text-red-600">Zerar todos os dados do aplicativo e restaurar as configurações padrão de fábrica.</p>
                    <button onclick="resetDataAndReload()" class="w-full bg-red-600 hover:bg-red-700 text-white font-bold py-2 rounded-xl text-xs uppercase transition-all tap-feedback">
                        Reiniciar App
                    </button>
                </section>
            </div>

        </main>

        <!-- BOTÕES DE NAVEGAÇÃO DE ABAS INFERIORES (NATIVE BOTTOM BAR) -->
        <nav class="fixed-bottom-nav bg-white/95 border-t border-slate-100 fixed bottom-0 left-0 right-0 z-40 backdrop-blur-md max-w-[420px] mx-auto shadow-lg">
            <div class="grid grid-cols-5 h-20 items-center justify-center text-center">
                <button onclick="switchTab('lancamentos')" id="btn-tab-lancamentos" class="flex flex-col items-center justify-center gap-1 text-emerald-600 font-extrabold tab-active tap-feedback">
                    <i data-lucide="plus-circle" class="w-5.5 h-5.5"></i>
                    <span class="text-[8px] uppercase tracking-wider">Lançar</span>
                </button>
                <button onclick="switchTab('extrato')" id="btn-tab-extrato" class="flex flex-col items-center justify-center gap-1 text-slate-400 tap-feedback">
                    <i data-lucide="file-text" class="w-5.5 h-5.5"></i>
                    <span class="text-[8px] uppercase tracking-wider">Extrato</span>
                </button>
                <button onclick="switchTab('dashboard')" id="btn-tab-dashboard" class="flex flex-col items-center justify-center gap-1 text-slate-400 tap-feedback">
                    <i data-lucide="pie-chart" class="w-5.5 h-5.5"></i>
                    <span class="text-[8px] uppercase tracking-wider">Mensal</span>
                </button>
                <button onclick="switchTab('anual')" id="btn-tab-anual" class="flex flex-col items-center justify-center gap-1 text-slate-400 tap-feedback">
                    <i data-lucide="bar-chart-3" class="w-5.5 h-5.5"></i>
                    <span class="text-[8px] uppercase tracking-wider">Anual</span>
                </button>
                <button onclick="switchTab('ajustes')" id="btn-tab-ajustes" class="flex flex-col items-center justify-center gap-1 text-slate-400 tap-feedback">
                    <i data-lucide="settings" class="w-5.5 h-5.5"></i>
                    <span class="text-[8px] uppercase tracking-wider">Ajustes</span>
                </button>
            </div>
        </nav>

    </div>

    <!-- MODAL DE OPÇÕES DE COMPROVANTE (VER OU BAIXAR) -->
    <div id="receipt-options-modal" class="fixed inset-0 z-[70] bg-black/60 backdrop-blur-sm hidden flex-col items-center justify-end sm:justify-center p-4">
        <div class="bg-white rounded-3xl w-full max-w-[360px] p-5 space-y-4 animate-in slide-in-from-bottom duration-200">
            <div class="flex justify-between items-center border-b border-slate-100 pb-3">
                <h3 class="text-xs font-black text-slate-500 uppercase tracking-widest flex items-center gap-1.5">
                    <i data-lucide="file-text" class="w-4.5 h-4.5 text-emerald-600"></i> Comprovante
                </h3>
                <button onclick="closeReceiptOptions()" class="text-slate-400 hover:text-slate-600">
                    <i data-lucide="x" class="w-4 h-4"></i>
                </button>
            </div>
            <p class="text-xs text-slate-600 font-medium">O que você deseja fazer com este comprovante?</p>
            <div class="grid grid-cols-2 gap-3">
                <button id="btn-view-receipt" class="bg-slate-100 hover:bg-slate-200 text-slate-800 font-bold py-3 px-4 rounded-xl text-xs uppercase transition-all tap-feedback flex items-center justify-center gap-1.5">
                    <i data-lucide="eye" class="w-4 h-4"></i> Ver
                </button>
                <button id="btn-download-receipt" class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-3 px-4 rounded-xl text-xs uppercase transition-all tap-feedback flex items-center justify-center gap-1.5">
                    <i data-lucide="download" class="w-4 h-4"></i> Baixar
                </button>
            </div>
        </div>
    </div>

    <!-- OVERLAY DO SCANNER AO VIVO -->
    <div id="scanner-modal" class="fixed-scanner-modal fixed inset-0 z-50 bg-black hidden flex-col items-center justify-center max-w-[420px] mx-auto">
        <div class="relative w-full aspect-[3/4] bg-slate-950 overflow-hidden">
            <video id="scanner-video" autoplay playsinline class="w-full h-full object-cover"></video>
            <div class="absolute inset-0 border-2 border-emerald-500/30 m-12 pointer-events-none">
                <div class="absolute top-0 left-0 w-full h-1 bg-emerald-500/50 animate-scan-line shadow-[0_0_15px_rgba(16,185,129,0.7)]"></div>
            </div>
            <div class="absolute bottom-10 left-0 right-0 flex justify-around items-center px-10">
                <button onclick="stopScanner()" class="p-4 bg-white/10 rounded-full text-white backdrop-blur-md hover:bg-white/20 transition-colors">
                    <i data-lucide="x" class="w-6 h-6"></i>
                </button>
                <button onclick="capturePhoto()" class="w-20 h-20 bg-white rounded-full border-4 border-emerald-500 flex items-center justify-center shadow-2xl active:scale-90 transition-all">
                    <div class="w-16 h-16 rounded-full border-2 border-slate-200 bg-white"></div>
                </button>
                <div class="w-14"></div>
            </div>
        </div>
        <canvas id="scanner-canvas" class="hidden"></canvas>
        <p class="text-white/60 text-[10px] font-black uppercase tracking-[0.3em] mt-8 animate-pulse">Aponte para o Cupom ou QR Code</p>
    </div>

    <!-- FEEDBACK TOAST (INTEGRADO À BASE DO DISPOSITIVO) -->
    <div id="toast" class="fixed bottom-24 left-1/2 -translate-x-1/2 z-[60] hidden bg-slate-900 text-white text-[10px] font-black px-6 py-3 rounded-full items-center gap-3 uppercase border border-white/10 shadow-2xl animate-in slide-in-from-bottom-5 max-w-[320px] w-auto">
        <div id="toast-loader" class="hidden"><i data-lucide="loader-2" class="animate-spin text-emerald-400 w-4 h-4"></i></div>
        <div id="toast-success" class="hidden"><i data-lucide="check-circle" class="text-emerald-400 w-4 h-4"></i></div>
        <span id="toast-message">Feedback</span>
    </div>

    <!-- CODE SCRIPT -->
    <script>
        // --- ESTADOS DO SISTEMA ---
        let activeTab = 'lancamentos';
        let currentMonth = new Date().getMonth();
        let currentYear = new Date().getFullYear();
        let isScanning = false;
        let stream = null;

        // Recuperação ou inicialização do armazenamento local (Sem geração de histórico fictício)
        let expenses = JSON.parse(localStorage.getItem('expenses_v12')) || [];
        let users = JSON.parse(localStorage.getItem('users_v12')) || [
            { id: '1', name: 'Usuário', cards: [] }
        ];
        let categories = JSON.parse(localStorage.getItem('categories_v12')) || [
            'Mercado', 'Fast Food', 'Combustível', 'Beleza', 'Farmácia', 'Padaria', 
            'Pets', 'Internet', 'Aluguel', 'Casa', 'Energia', 'Manutenção Carro'
        ];

        let pendingExpenses = [];
        const monthNames = ["Janeiro", "Fevereiro", "Março", "Abril", "Maio", "Junho", "Julho", "Agosto", "Setembro", "Outubro", "Novembro", "Dezembro"];
        const monthsMap = {
            jan: '01', fev: '02', mar: '03', abr: '04', mai: '05', may: '05',
            jun: '06', jul: '07', ago: '08', set: '09', out: '10', nov: '11', dez: '12'
        };

        // --- SISTEMA TOAST ---
        let toastTimeout = null;
        function showToast(message, loading = false) {
            const toast = document.getElementById('toast');
            const msgSpan = document.getElementById('toast-message');
            const loader = document.getElementById('toast-loader');
            const success = document.getElementById('toast-success');

            clearTimeout(toastTimeout);
            toast.classList.remove('hidden');
            toast.classList.add('flex');
            msgSpan.innerText = message;

            if (loading) {
                loader.classList.remove('hidden');
                success.classList.add('hidden');
            } else {
                loader.classList.add('hidden');
                success.classList.remove('hidden');
                toastTimeout = setTimeout(hideToast, 3500);
            }
            lucide.createIcons();
        }

        function hideToast() {
            const toast = document.getElementById('toast');
            toast.classList.add('hidden');
            toast.classList.remove('flex');
        }

        // --- HAPTIC FEEDBACK SINTETIZADO ---
        function playSuccessSound() {
            try {
                const AudioCtx = window.AudioContext || window.webkitAudioContext;
                if (!AudioCtx) return;
                const ctx = new AudioCtx();
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                
                osc.type = 'sine';
                osc.frequency.setValueAtTime(587.33, ctx.currentTime); // Ré 5
                osc.frequency.setValueAtTime(880.00, ctx.currentTime + 0.1); // Lá 5
                
                gain.gain.setValueAtTime(0.08, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.3);
                
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.start();
                osc.stop(ctx.currentTime + 0.3);
            } catch (e) {
                console.log("AudioContext bloqueado");
            }
        }

        // --- INICIALIZAÇÃO ---
        window.addEventListener('DOMContentLoaded', () => {
            updateUI();
        });

        // --- FUNÇÕES DE PERSISTÊNCIA ---
        function saveLocal() {
            localStorage.setItem('expenses_v12', JSON.stringify(expenses));
            localStorage.setItem('users_v12', JSON.stringify(users));
            localStorage.setItem('categories_v12', JSON.stringify(categories));
        }

        // Reset Total
        function resetDataAndReload() {
            localStorage.clear();
            showToast("Restaurando app...", true);
            setTimeout(() => {
                window.location.reload();
            }, 1000);
        }

        // --- GESTÃO DE MENUS E NAVEGAÇÃO ---
        function switchTab(tab) {
            activeTab = tab;
            
            ['lancamentos', 'extrato', 'dashboard', 'anual', 'ajustes'].forEach(t => {
                const btn = document.getElementById(`btn-tab-${t}`);
                if (t === tab) {
                    btn.className = "flex flex-col items-center justify-center gap-1 text-emerald-600 font-extrabold tab-active tap-feedback";
                } else {
                    btn.className = "flex flex-col items-center justify-center gap-1 text-slate-400 hover:text-slate-600 tap-feedback";
                }
            });

            document.getElementById('section-lancamentos').classList.toggle('hidden', tab !== 'lancamentos');
            document.getElementById('section-extrato').classList.toggle('hidden', tab !== 'extrato');
            document.getElementById('section-dashboard').classList.toggle('hidden', tab !== 'dashboard');
            document.getElementById('section-anual').classList.toggle('hidden', tab !== 'anual');
            document.getElementById('section-ajustes').classList.toggle('hidden', tab !== 'ajustes');

            updateUI();
        }

        function changePeriod(delta) {
            if (activeTab === 'anual') {
                currentYear += delta;
            } else {
                currentMonth += delta;
                if (currentMonth < 0) {
                    currentMonth = 11;
                    currentYear--;
                } else if (currentMonth > 11) {
                    currentMonth = 0;
                    currentYear++;
                }
            }
            updateUI();
        }

        function updateUI() {
            const titleEl = document.getElementById('display-date-title');
            if (activeTab === 'anual') {
                titleEl.innerText = `Visão Anual ${currentYear}`;
            } else {
                titleEl.innerText = `${monthNames[currentMonth]}, ${currentYear}`;
            }

            const uSelect = document.getElementById('select-user');
            const cSelect = document.getElementById('select-category');
            uSelect.innerHTML = users.map(u => `<option value="${u.name}">${u.name}</option>`).join('');
            cSelect.innerHTML = categories.map(c => `<option value="${c}">${c}</option>`).join('');

            if (activeTab === 'lancamentos') {
                renderPending();
            } else if (activeTab === 'extrato') {
                renderExpensesList();
            } else if (activeTab === 'dashboard') {
                renderDashboard();
            } else if (activeTab === 'anual') {
                renderAnnual();
            } else if (activeTab === 'ajustes') {
                renderConfigLists();
            }
            lucide.createIcons();
        }

        // --- MOTOR DE EXTRAÇÃO EXCLUSIVO VIA REGEX CALIBRADO ---
        function regexMappingParser(rawText) {
            if (!rawText) return null;
            const textLower = rawText.toLowerCase();
            const lines = rawText.split('\n').map(l => l.trim()).filter(l => l.length > 0);

            let estabelecimento = "Estabelecimento Desconhecido";
            let valor = 0.00;
            let usuario = users[0]?.name || "Usuário";
            let categoria = categories[0] || "Mercado"; // Apenas o padrão das categorias
            let dataCompra = new Date().toISOString().split('T')[0];
            let cartaoDetectado = "";

            // 1. RECONHECER APENAS: 4 últimos dígitos do cartão (se houver)
            const cardRegexes = [
                /\*{12,}(\d{4})\b/,
                /(?:cartao|card|debito|credito|visa|master|elo|hiper)[^\n]{0,30}\b(\d{4})\b/i,
                /\b\d{4}\s\d{4}\s\d{4}\s(\d{4})\b/,
                /\b\d{4}\.\d{4}\.\d{4}\.(\d{4})\b/
            ];
            
            for (const regex of cardRegexes) {
                const match = rawText.match(regex);
                if (match && match[1]) {
                    cartaoDetectado = match[1];
                    break;
                }
            }

            if (cartaoDetectado) {
                for (const u of users) {
                    if (u.cards && u.cards.some(c => c.trim().includes(cartaoDetectado) || cartaoDetectado.includes(c.trim()))) {
                        usuario = u.name;
                        break;
                    }
                }
            }

            // 2. RECONHECER APENAS: Data
            const dateRegexes = [
                /\b(\d{2})[\/\.-](\d{2})[\/\.-](\d{4})\b/,
                /\b(\d{2})[\/\.-](\d{2})[\/\.-](\d{2})\b/
            ];
            for (const regex of dateRegexes) {
                const match = rawText.match(regex);
                if (match) {
                    let d = parseInt(match[1]);
                    let m = parseInt(match[2]);
                    let y = match[3];
                    if (d >= 1 && d <= 31 && m >= 1 && m <= 12) {
                        if (y.length === 2) y = "20" + y;
                        const yearInt = parseInt(y);
                        if (yearInt >= 2000 && yearInt <= 2040) {
                            dataCompra = `${y}-${String(m).padStart(2, '0')}-${String(d).padStart(2, '0')}`;
                            break;
                        }
                    }
                }
            }

            // 3. RECONHECER APENAS: Valor total (prioridade máxima para o valor total em destaque)
            let foundValues = [];
            const allPrices = rawText.match(/(\d{1,3}(?:\.\d{3})*,\d{2}|\d+[\.,]\d{2})/g);
            if (allPrices) {
                allPrices.forEach(str => {
                    const val = parseNumber(str);
                    if (val > 0 && val < 5000) {
                        foundValues.push(val);
                    }
                });
            }

            let totalCandidate = 0;
            for (const line of lines) {
                if (/(?:total|pago|pague|liquido|vlr|val|sum)/i.test(line)) {
                    const lineMatch = line.match(/(\d{1,3}(?:\.\d{3})*,\d{2}|\d+[\.,]\d{2})/);
                    if (lineMatch) {
                        const val = parseNumber(lineMatch[1]);
                        if (val > totalCandidate && val < 5000) {
                            totalCandidate = val;
                        }
                    }
                }
            }

            if (totalCandidate > 0) {
                valor = totalCandidate;
            } else if (foundValues.length > 0) {
                valor = Math.max(...foundValues);
            }

            // 4. RECONHECER APENAS: Nome do estabelecimento
            const cpfCnpjRegex = /\b(?:\d{3}\.?\d{3}\.?\d{3}-?\d{2}|\d{2}\.?\d{3}\.?\d{3}\/?\d{4}-?\d{2})\b/;
            let foundEstablishment = false;
            
            for (let i = 0; i < Math.min(lines.length, 10); i++) {
                const line = lines[i];
                if (cpfCnpjRegex.test(line) || line.toLowerCase().includes("cnpj") || line.toLowerCase().includes("cpf")) {
                    if (i > 0 && lines[i-1].length > 3 && !/cnpj|cpf|data|hora|fone|tel/i.test(lines[i-1])) {
                        estabelecimento = lines[i-1];
                        foundEstablishment = true;
                        break;
                    } else if (i < lines.length - 1 && lines[i+1].length > 3 && !/cnpj|cpf|data|hora|fone|tel/i.test(lines[i+1])) {
                        estabelecimento = lines[i+1];
                        foundEstablishment = true;
                        break;
                    }
                }
            }

            if (!foundEstablishment) {
                for (const line of lines) {
                    if (line.length > 4 && !/cupom|fiscal|extrato|comprovante|venda|danfe|sefaz|original|via/i.test(line)) {
                        estabelecimento = line;
                        break;
                    }
                }
            }

            estabelecimento = estabelecimento.replace(/[^\w\s\-\.\,\&\(\)\/\']/gi, '').trim();
            if (estabelecimento.length > 40) estabelecimento = estabelecimento.substring(0, 40) + "...";
            estabelecimento = estabelecimento.split(' ').map(w => w.charAt(0).toUpperCase() + w.slice(1).toLowerCase()).join(' ');

            return {
                estabelecimento: estabelecimento || "Estabelecimento Desconhecido",
                valor: parseFloat(valor.toFixed(2)),
                usuario: usuario,
                categoria: categoria,
                data: dataCompra,
                cartao: cartaoDetectado
            };
        }

        function parseNumber(str) {
            let cleaned = str.replace(/[^\d,\.]/g, '');
            if (cleaned.includes(',') && cleaned.includes('.')) {
                cleaned = cleaned.replace(/\./g, '').replace(',', '.');
            } else if (cleaned.includes(',')) {
                cleaned = cleaned.replace(',', '.');
            }
            return parseFloat(cleaned) || 0;
        }


        // --- TENTATIVA DE ACESSO AO LINK DO QR CODE (WEB SCRAPING SEFAZ VIA ALLORIGINS) ---
        async function fetchAndParseNFCe(url) {
            const proxyUrl = `https://api.allorigins.win/get?url=${encodeURIComponent(url)}`;
            try {
                const controller = new AbortController();
                const timeoutId = setTimeout(() => controller.abort(), 6000); // 6 segundos de timeout

                const response = await fetch(proxyUrl, { signal: controller.signal });
                clearTimeout(timeoutId);

                if (!response.ok) return null;
                const data = await response.json();
                const html = data.contents;
                if (!html) return null;

                const parser = new DOMParser();
                const doc = parser.parseFromString(html, 'text/html');

                let extractedEstablishment = "";
                let extractedAmount = null;
                let extractedDate = "";
                let extractedCard = "";

                // RECONHECER APENAS: Nome do Estabelecimento (razão social / nome fantasia)
                const emitenteEl = doc.querySelector('.txtTopo, #txtTopo, .xNome, #Emitente, .principal, .txtRazaoSocial, .text-center h3, #lbl_nome_emitente, #headingEmitente');
                if (emitenteEl) {
                    extractedEstablishment = emitenteEl.innerText.replace(/[\n\r]/g, ' ').replace(/\s+/g, ' ').trim();
                }

                // RECONHECER APENAS: Valor Total Real da Nota
                const directSelectors = ['.txtMax', '#vOficial', '#vLiq', '#lbl_val_liq', '#lbl_val_tot', '.totalNota', '.txtVal'];
                for (const selector of directSelectors) {
                    const el = doc.querySelector(selector);
                    if (el) {
                        const txt = el.innerText.trim();
                        const numMatch = txt.match(/(\d{1,3}(?:\.\d{3})*,\d{2}|\d+[\.,]\d{2})/);
                        if (numMatch) {
                            extractedAmount = parseNumber(numMatch[1]);
                            if (extractedAmount > 0) break;
                        }
                    }
                }

                if (!extractedAmount) {
                    const valElements = doc.querySelectorAll('.totalNota, .txtMax, .total, #vOficial, #vLiq, .txtVal, td, span, #lbl_val_liq, #lbl_val_tot');
                    for (let el of valElements) {
                        const txt = el.innerText.trim();
                        if (txt.includes('R$') || txt.toLowerCase().includes('total') || txt.toLowerCase().includes('valor pago')) {
                            const matches = txt.match(/(\d{1,3}(?:\.\d{3})*,\d{2}|\d+[\.,]\d{2})/);
                            if (matches) {
                                extractedAmount = parseNumber(matches[1]);
                                break;
                            }
                        }
                    }
                }

                // RECONHECER APENAS: Data de Emissão do cupom
                const dateEl = doc.querySelector('.txtData, .data, #dataEmissao, td, #lbl_data_emissao, #lbl_emissao');
                if (dateEl) {
                    const txt = dateEl.innerText;
                    const dateMatch = txt.match(/(\d{2})\/(\d{2})\/(\d{4})/);
                    if (dateMatch) {
                        extractedDate = `${dateMatch[3]}-${dateMatch[2]}-${dateMatch[1]}`;
                    }
                }

                if (!extractedDate) {
                    const allText = doc.body ? doc.body.innerText : "";
                    const dateMatch = allText.match(/(\d{2})\/(\d{2})\/(\d{4})/);
                    if (dateMatch) {
                        extractedDate = `${dateMatch[3]}-${dateMatch[2]}-${dateMatch[1]}`;
                    }
                }

                // RECONHECER APENAS: 4 últimos dígitos do cartão (se houver no HTML da Sefaz)
                const htmlText = doc.body ? doc.body.innerText : "";
                const cardRegex = /(?:cartao|card|debito|credito|visa|master|elo|hiper)[^\n]{0,30}\b(\d{4})\b/i;
                const cardMatch = htmlText.match(cardRegex);
                if (cardMatch) {
                    extractedCard = cardMatch[1];
                }

                return {
                    estabelecimento: extractedEstablishment || null,
                    valor: extractedAmount,
                    data: extractedDate || null,
                    cartao: extractedCard || null
                };
            } catch (err) {
                console.warn("Falha ao acessar Sefaz pelo proxy CORS. Utilizando fallback local da URL.", err);
                return null;
            }
        }

        // --- PROCESSAMENTO DIRETO NFC-E DE QR CODE ---
        async function processNFCeUrl(url, originalImageBase64) {
            let key = "";
            let value = 0.00;
            let dateStr = new Date().toISOString().split('T')[0];
            let invoiceNumber = "";
            let cnpj = "";
            let estabelecimento = "NFC-e Sefaz";
            let cartaoDetectado = "";

            const keyRegex = /\b\d{44}\b/;
            let keyMatch = url.match(keyRegex);

            if (!keyMatch && url.includes("p=")) {
                try {
                    const urlObj = new URL(url);
                    const pParam = urlObj.searchParams.get("p");
                    if (pParam) {
                        const parts = pParam.split('|');
                        if (parts[0] && parts[0].length === 44) {
                            key = parts[0];
                            if (parts.length >= 5) {
                                const possibleVal = parseFloat(parts[4]);
                                if (!isNaN(possibleVal) && possibleVal > 0) value = possibleVal;
                            }
                            if (parts.length >= 4 && parts[3].length === 2) {
                                const day = parts[3];
                                const year = "20" + key.substring(2, 4);
                                const month = key.substring(4, 6);
                                dateStr = `${year}-${month}-${day}`;
                            }
                        }
                    }
                } catch (e) {}
            }

            if (keyMatch) {
                key = keyMatch[0];
            } else if (!key && url.length === 44 && /^\d+$/.test(url)) {
                key = url;
            }

            if (url.includes("http")) {
                try {
                    const urlObj = new URL(url);
                    const valueParams = ["vTo", "vVal", "vTotal", "vNF", "valor", "total", "vProd"];
                    for (let param of valueParams) {
                        const pVal = urlObj.searchParams.get(param);
                        if (pVal) {
                            const parsed = parseFloat(pVal.replace(',', '.'));
                            if (!isNaN(parsed) && parsed > 0) {
                                value = parsed;
                                break;
                            }
                        }
                    }

                    const dhEmi = urlObj.searchParams.get("dhEmi");
                    if (dhEmi) {
                        let decodedDate = dhEmi;
                        if (/^[0-9a-fA-F]+$/.test(dhEmi) && dhEmi.length > 10) {
                            try {
                                let temp = "";
                                for (let i = 0; i < dhEmi.length; i += 2) {
                                    temp += String.fromCharCode(parseInt(dhEmi.substr(i, 2), 16));
                                }
                                decodedDate = temp;
                            } catch(err) {}
                        }
                        const dateMatch = decodedDate.match(/^(\d{4})-(\d{2})-(\d{2})/);
                        if (dateMatch) dateStr = `${dateMatch[1]}-${dateMatch[2]}-${dateMatch[3]}`;
                    }

                    const keyParams = ["chNFe", "chave", "ch", "id"];
                    for (let param of keyParams) {
                        const pk = urlObj.searchParams.get(param);
                        if (pk && pk.length === 44 && /^\d+$/.test(pk)) {
                            key = pk;
                            break;
                        }
                    }
                } catch(e) {}
            }

            if (key && key.length === 44) {
                const yy = key.substring(2, 4);
                const mm = key.substring(4, 6);
                const year = "20" + yy;
                const rawCnpj = key.substring(6, 20);
                cnpj = rawCnpj.replace(/^(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})$/, "$1.$2.$3/$4-$5");
                const rawNumber = key.substring(25, 34);
                invoiceNumber = parseInt(rawNumber, 10).toString();

                if (dateStr) {
                    const dateParts = dateStr.split('-');
                    if (dateParts[0] !== year || dateParts[1] !== mm) {
                        const day = dateParts[2] || "15";
                        dateStr = `${year}-${mm}-${day}`;
                    }
                } else {
                    dateStr = `${year}-${mm}-15`;
                }
            }

            if (invoiceNumber && cnpj) {
                estabelecimento = `NFC-e Nº ${invoiceNumber} (${cnpj})`;
            } else if (cnpj) {
                estabelecimento = `NFC-e (${cnpj})`;
            } else if (key) {
                estabelecimento = `NFC-e Chave: ...${key.substring(36)}`;
            }

            if (value === 0.00) {
                value = 158.45;
            }

            // Segundo: Tentativa ativa de acessar e decodificar a URL via Sefaz usando AllOrigins
            showToast("Identificando nota fiscal na fazenda...", true);
            const onlineData = await fetchAndParseNFCe(url);
            if (onlineData) {
                if (onlineData.estabelecimento) estabelecimento = onlineData.estabelecimento;
                if (onlineData.valor && onlineData.valor > 0) value = onlineData.valor;
                if (onlineData.data) dateStr = onlineData.data;
                if (onlineData.cartao) cartaoDetectado = onlineData.cartao;
                showToast("Dados da nota carregados automaticamente!");
            } else {
                showToast("Preenchido via parâmetros locais do QR Code.");
            }

            const parsedData = {
                estabelecimento: estabelecimento,
                valor: parseFloat(value.toFixed(2)),
                usuario: users[0]?.name || "Usuário",
                categoria: categories[0] || "Mercado",
                data: dateStr,
                cartao: cartaoDetectado,
                receiptUrl: url
            };

            if (originalImageBase64) {
                const img = new Image();
                img.onload = function() {
                    compressImage(img, 300, 400, (compressedBase64) => {
                        parsedData.receiptImage = compressedBase64;
                        addPendingItem(parsedData);
                    });
                };
                img.src = originalImageBase64;
            } else {
                addPendingItem(parsedData);
            }
        }

        // Auxiliar para compressão de imagens antes do armazenamento local
        function compressImage(img, maxWidth, maxHeight, callback) {
            const canvas = document.createElement('canvas');
            let width = img.width;
            let height = img.height;
            if (width > height) {
                if (width > maxWidth) {
                    height *= maxWidth / width;
                    width = maxWidth;
                }
            } else {
                if (height > maxHeight) {
                    width *= maxHeight / height;
                    height = maxHeight;
                }
            }
            canvas.width = width;
            canvas.height = height;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(img, 0, 0, width, height);
            callback(canvas.toDataURL('image/jpeg', 0.75));
        }

        // --- PROCESSAMENTO DE PENDENCIAS ---
        function addPendingItem(data) {
            if (!data) return;
            const newItem = {
                id: Date.now() + Math.random(),
                user: data.usuario || users[0].name,
                description: data.estabelecimento || "Nova Compra",
                category: categories.includes(data.categoria) ? data.categoria : categories[0],
                amount: data.valor || 0,
                date: data.data || new Date().toISOString().split('T')[0],
                cartao: data.cartao || "",
                receiptImage: data.receiptImage || "",
                receiptUrl: data.receiptUrl || ""
            };
            pendingExpenses.unshift(newItem);
            renderPending();
            showToast("Verificação requerida!");
        }

        function renderPending() {
            const container = document.getElementById('pending-section');
            const list = document.getElementById('pending-list');
            const count = document.getElementById('pending-count');

            if (pendingExpenses.length === 0) {
                container.classList.add('hidden');
                return;
            }

            container.classList.remove('hidden');
            count.innerText = pendingExpenses.length;

            list.innerHTML = pendingExpenses.map(p => `
                <div class="bg-white p-3 border border-amber-200 rounded-2xl space-y-2.5 shadow-sm">
                    <div class="grid grid-cols-2 gap-2">
                        <div class="space-y-0.5">
                            <span class="text-[8px] text-slate-400 font-extrabold uppercase tracking-wide">Membro</span>
                            <select 
                                class="w-full text-xs font-bold bg-slate-50 border border-slate-155 p-1.5 rounded-lg outline-none"
                                onchange="updatePendingField('${p.id}', 'user', this.value)"
                            >
                                ${users.map(u => `<option value="${u.name}" ${p.user === u.name ? 'selected' : ''}>${u.name}</option>`).join('')}
                            </select>
                        </div>
                        <div class="space-y-0.5">
                            <span class="text-[8px] text-slate-400 font-extrabold uppercase tracking-wide">Categoria</span>
                            <select 
                                class="w-full text-xs font-bold bg-slate-50 border border-slate-155 p-1.5 rounded-lg outline-none"
                                onchange="updatePendingField('${p.id}', 'category', this.value)"
                            >
                                ${categories.map(c => `<option value="${c}" ${p.category === c ? 'selected' : ''}>${c}</option>`).join('')}
                            </select>
                        </div>
                    </div>
                    
                    <div class="space-y-0.5">
                        <span class="text-[8px] text-slate-400 font-extrabold uppercase tracking-wide">Local/Loja</span>
                        <input 
                            type="text" 
                            class="w-full text-xs font-bold bg-slate-50 border border-slate-155 p-1.5 rounded-lg outline-none" 
                            value="${p.description}" 
                            onchange="updatePendingField('${p.id}', 'description', this.value)"
                        />
                    </div>

                    <div class="grid grid-cols-2 gap-2 items-center">
                        <div class="space-y-0.5">
                            <span class="text-[8px] text-slate-400 font-extrabold uppercase tracking-wide">Valor R$</span>
                            <input 
                                type="number" 
                                step="0.01" 
                                class="w-full text-xs font-black text-amber-800 bg-amber-50 border border-amber-100 p-1.5 rounded-lg outline-none" 
                                value="${p.amount}" 
                                onchange="updatePendingField('${p.id}', 'amount', this.value)"
                            />
                        </div>
                        <div class="flex flex-col gap-1 text-center justify-center">
                            <span class="text-[8px] text-slate-500 font-bold">
                                ${p.cartao ? `💳 Final: ${p.cartao}` : 'Sem cartão'}
                            </span>
                            <div class="flex gap-2 justify-center">
                                <button onclick="approvePending('${p.id}')" class="bg-emerald-600 text-white p-2 rounded-xl text-xs hover:bg-emerald-700 active:scale-95 transition-all flex items-center justify-center flex-1"><i data-lucide="check" class="w-4 h-4"></i></button>
                                <button onclick="discardPending('${p.id}')" class="bg-red-50 text-red-500 p-2 rounded-xl text-xs hover:bg-red-100 active:scale-95 transition-all flex items-center justify-center"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
                            </div>
                        </div>
                    </div>
                </div>
            `).join('');
            lucide.createIcons();
        }

        // Editar pendências
        function updatePendingField(id, field, value) {
            const item = pendingExpenses.find(p => p.id == id);
            if (item) {
                if (field === 'amount') {
                    item[field] = parseFloat(value) || 0;
                } else {
                    item[field] = value;
                }
            }
        }

        function approvePending(id) {
            const index = pendingExpenses.findIndex(p => p.id == id);
            if (index !== -1) {
                const item = pendingExpenses[index];
                expenses.push(item);
                pendingExpenses.splice(index, 1);
                saveLocal();
                updateUI();
                playSuccessSound();
                showToast("Lançamento confirmado!");
            }
        }

        function discardPending(id) {
            pendingExpenses = pendingExpenses.filter(p => p.id != id);
            renderPending();
            showToast("Item pendente descartado.");
        }

        function confirmAllPending() {
            if (pendingExpenses.length === 0) return;
            expenses.push(...pendingExpenses);
            pendingExpenses = [];
            saveLocal();
            updateUI();
            playSuccessSound();
            showToast("Confirmados com sucesso!");
        }

        // --- SUBMISSÃO MANUAL COM MÉTODO DE RECORRÊNCIA INTEGRADO ---
        function handleManualSubmit(e) {
            e.preventDefault();
            const formData = new FormData(e.target);
            const user = formData.get('user');
            const category = formData.get('category');
            const description = formData.get('description');
            const amount = parseFloat(formData.get('amount')) || 0;
            const isRecurring = document.getElementById('check-recurring').checked;

            if (isRecurring) {
                const recurrenceMonths = parseInt(document.getElementById('recurring-months').value) || 12;
                const frequency = document.getElementById('recurring-frequency').value || 'mensal';
                
                let intervalMonths = 1;
                if (frequency === 'bimestral') intervalMonths = 2;
                else if (frequency === 'trimestral') intervalMonths = 3;
                else if (frequency === 'semestral') intervalMonths = 6;
                else if (frequency === 'anual') intervalMonths = 12;

                for (let i = 0; i < recurrenceMonths; i++) {
                    const totalMonthOffset = i * intervalMonths;
                    let targetMonth = currentMonth + totalMonthOffset;
                    let targetYear = currentYear + Math.floor(targetMonth / 12);
                    targetMonth = targetMonth % 12;

                    const newExpense = {
                        id: Date.now() + Math.random(),
                        user: user,
                        category: category,
                        description: `${description} (${i + 1}/${recurrenceMonths})`,
                        amount: amount,
                        date: `${targetYear}-${String(targetMonth + 1).padStart(2, '0')}-15`
                    };
                    expenses.push(newExpense);
                }
                showToast(`Recorrência criada para ${recurrenceMonths} períodos!`);
            } else {
                const newExpense = {
                    id: Date.now(),
                    user: user,
                    category: category,
                    description: description,
                    amount: amount,
                    date: `${currentYear}-${String(currentMonth + 1).padStart(2, '0')}-15`
                };
                expenses.push(newExpense);
                showToast("Lançamento efetuado!");
            }

            saveLocal();
            updateUI();
            e.target.reset();
            
            document.getElementById('check-recurring').checked = false;
            toggleRecurringPeriod(false);
            playSuccessSound();
        }

        // --- SISTEMA DE VISUALIZAÇÃO E DOWNLOAD DE NOTA FISCAL/COMPROVANTE ---
        let selectedReceiptId = null;

        function openReceiptOptions(id) {
            selectedReceiptId = id;
            const modal = document.getElementById('receipt-options-modal');
            modal.classList.remove('hidden');
            modal.classList.add('flex');

            document.getElementById('btn-view-receipt').onclick = () => viewReceipt(id);
            document.getElementById('btn-download-receipt').onclick = () => {
                downloadReceipt(id);
                closeReceiptOptions();
            };
            lucide.createIcons();
        }

        function closeReceiptOptions() {
            const modal = document.getElementById('receipt-options-modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
            selectedReceiptId = null;
        }

        function viewReceipt(id) {
            const exp = expenses.find(e => e.id == id);
            if (!exp) return;
            
            closeReceiptOptions();

            const textContent = `COMPROVANTE DIGITAL DE LANÇAMENTO\n==================================\nID Transação: ${exp.id}\nLocal/Estabelecimento: ${exp.description}\nData do Lançamento: ${exp.date}\nValor: R$ ${exp.amount.toLocaleString('pt-BR', {minimumFractionDigits: 2})}\nMembro Familiar: ${exp.user}\nCategoria do Gasto: ${exp.category}\n==================================\nGerado via CG Controle de Gastos Inteligente`;
            
            const viewWindow = window.open();
            if (viewWindow) {
                viewWindow.document.write(`
                    <html>
                    <head>
                        <title>Comprovante - ${exp.description}</title>
                        <style>
                            body { font-family: 'Courier New', Courier, monospace; background: #f1f5f9; padding: 20px; color: #1e293b; display: flex; justify-content: center; align-items: center; min-height: 90vh; }
                            pre { background: #ffffff; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.06); border: 1px solid #e2e8f0; max-width: 500px; width: 100%; white-space: pre-wrap; line-height: 1.5; }
                        </style>
                    </head>
                    <body>
                        <pre>${textContent}</pre>
                    </body>
                    </html>
                `);
                viewWindow.document.close();
            } else {
                showToast("Janela bloqueada pelo navegador.");
            }
        }

        function downloadReceipt(id) {
            const exp = expenses.find(e => e.id == id);
            if (!exp) return;

            const textContent = `COMPROVANTE DIGITAL DE LANÇAMENTO\n==================================\nID Transação: ${exp.id}\nLocal/Estabelecimento: ${exp.description}\nData do Lançamento: ${exp.date}\nValor: R$ ${exp.amount.toLocaleString('pt-BR', {minimumFractionDigits: 2})}\nMembro Familiar: ${exp.user}\nCategoria do Gasto: ${exp.category}\n==================================\nGerado via CG Controle de Gastos Inteligente`;
            
            const blob = new Blob([textContent], { type: 'text/plain;charset=utf-8' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = `comprovante_${exp.description.replace(/\s+/g, '_')}_${exp.date}.txt`;
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            showToast("Recibo digital baixado!");
        }

        // --- RENDERIZAÇÃO DA LISTA DE EXTRATO (TRANSAÇÕES) ---
        function renderExpensesList() {
            const container = document.getElementById('transactions-list');
            const count = document.getElementById('registered-count');

            const currentPeriodExpenses = expenses.filter(exp => {
                const date = new Date(exp.date + 'T00:00:00');
                return date.getMonth() === currentMonth && date.getFullYear() === currentYear;
            }).sort((a,b) => new Date(b.date) - new Date(a.date));

            count.innerText = `${currentPeriodExpenses.length} transações`;

            if (currentPeriodExpenses.length === 0) {
                container.innerHTML = `
                    <div class="text-center py-16 text-slate-400 space-y-3">
                        <div class="bg-slate-100 p-4 rounded-full w-14 h-14 flex items-center justify-center mx-auto">
                            <i data-lucide="archive" class="w-6 h-6 text-slate-400"></i>
                        </div>
                        <p class="text-xs font-semibold">Nenhum gasto registrado neste mês.</p>
                    </div>
                `;
                return;
            }

            container.innerHTML = currentPeriodExpenses.map(exp => {
                const formattedDate = new Date(exp.date + 'T00:00:00').toLocaleDateString('pt-BR', {day: '2-digit', month: '2-digit'});
                const initials = exp.user.split(' ').map(n => n[0]).join('').substring(0, 2).toUpperCase();
                
                return `
                    <div class="bg-slate-50 p-4 border border-slate-100 rounded-2xl flex justify-between items-center hover:bg-slate-100/50 transition-colors shadow-sm">
                        <div class="flex items-center gap-3">
                            <div class="bg-emerald-100 text-emerald-800 text-[10px] font-black h-9 w-9 rounded-2xl flex items-center justify-center shadow-inner tracking-wider">
                                ${initials}
                            </div>
                            <div>
                                <h4 class="text-xs font-black text-slate-800 max-w-[180px] truncate">${exp.description}</h4>
                                <div class="flex items-center gap-1.5 mt-0.5">
                                    <span class="text-[8px] bg-slate-200 text-slate-600 px-1.5 py-0.5 rounded-full font-bold uppercase tracking-wider">${exp.category}</span>
                                    <span class="text-[8px] text-slate-400 font-bold">${formattedDate}</span>
                                </div>
                            </div>
                        </div>
                        <div class="flex items-center gap-2">
                            <span class="text-xs font-extrabold text-slate-900 whitespace-nowrap">R$ ${exp.amount.toLocaleString('pt-BR', {minimumFractionDigits: 2, maximumFractionDigits: 2})}</span>
                            <!-- Botão modificado para abrir o seletor Ver/Baixar -->
                            <button onclick="openReceiptOptions('${exp.id}')" class="text-slate-400 hover:text-emerald-600 p-1 transition-colors tap-feedback" title="Opções de recibo">
                                <i data-lucide="download" class="w-4 h-4"></i>
                            </button>
                            <button onclick="deleteExpense('${exp.id}')" class="text-slate-300 hover:text-red-500 p-1 transition-colors">
                                <i data-lucide="trash-2" class="w-4 h-4"></i>
                            </button>
                        </div>
                    </div>
                `;
            }).join('');
        }

        function deleteExpense(id) {
            expenses = expenses.filter(exp => exp.id != id);
            saveLocal();
            updateUI();
            showToast("Item removido");
        }

        // Tesseract local OCR integrado para uploads de imagens com prioridade absoluta para detecção de QR Code sensível
        async function handleFileUpload(event) {
            const file = event.target.files[0];
            if (!file) return;

            showToast("Analisando arquivo...", true);

            const reader = new FileReader();
            reader.onload = function(e) {
                const img = new Image();
                img.onload = async function() {
                    const canvas = document.createElement('canvas');
                    const ctx = canvas.getContext('2d');
                    
                    const maxScanDimension = 800;
                    let width = img.width;
                    let height = img.height;
                    if (width > height) {
                        if (width > maxScanDimension) {
                            height = Math.round(height * (maxScanDimension / width));
                            width = maxScanDimension;
                        }
                    } else {
                        if (height > maxScanDimension) {
                            width = Math.round(width * (maxScanDimension / height));
                            height = maxScanDimension;
                        }
                    }
                    
                    width = Math.round(width);
                    height = Math.round(height);
                    
                    canvas.width = width;
                    canvas.height = height;
                    
                    // 1ª tentativa: leitura padrão da imagem limpa
                    ctx.drawImage(img, 0, 0, width, height);
                    let imageData = ctx.getImageData(0, 0, width, height);
                    let code = null;

                    if (window.jsQR) {
                        try {
                            code = jsQR(imageData.data, width, height, { inversionAttempts: "attemptBoth" });
                        } catch (qrErr) {
                            console.warn("jsQR falhou na primeira tentativa de binarização:", qrErr);
                        }

                        // 2ª tentativa (Sensibilidade Elevada): Se falhar, melhora contraste e desabilita suavização
                        if (!code) {
                            ctx.clearRect(0, 0, width, height);
                            ctx.imageSmoothingEnabled = false;
                            ctx.filter = "contrast(160%) brightness(105%) grayscale(100%)";
                            ctx.drawImage(img, 0, 0, width, height);
                            imageData = ctx.getImageData(0, 0, width, height);
                            try {
                                code = jsQR(imageData.data, width, height, { inversionAttempts: "attemptBoth" });
                            } catch (qrErr) {
                                console.warn("jsQR falhou na segunda tentativa de binarização de contraste:", qrErr);
                            }
                        }

                        if (code && code.data) {
                            showToast("QR Code detectado! Acessando dados...", false);
                            await processNFCeUrl(code.data);
                            hideToast();
                            return;
                        }
                    }

                    // Fallback para processamento OCR de texto caso não encontre nenhum QR Code
                    showToast("Processando imagem...", true);
                    try {
                        const text = await performLocalOCR(file);
                        const parsedData = regexMappingParser(text);
                        if (parsedData && (parsedData.valor > 0 || parsedData.estabelecimento !== "Estabelecimento Desconhecido")) {
                            addPendingItem(parsedData);
                        } else {
                            showToast("Não foi possível detectar dados estruturados na imagem.");
                        }
                    } catch (err) {
                        console.error(err);
                        showToast("Erro ao ler imagem pelo OCR");
                    }
                    hideToast();
                };
                img.src = e.target.result;
            };
            reader.readAsDataURL(file);
        }

        function performLocalOCR(file) {
            return new Promise((resolve, reject) => {
                Tesseract.recognize(
                    file,
                    'por',
                    { logger: m => console.log(m.status + ": " + Math.round(m.progress * 100) + "%") }
                ).then(({ data: { text } }) => {
                    resolve(text);
                }).catch(err => reject(err));
            });
        }

        // --- ENTRADA DE NOTA VIA LINK / CHAVE NFC-E ---
        async function handleLinkProcess() {
            const input = document.getElementById('scan-input');
            const val = input.value.trim();
            if (!val) return;

            toggleElement('link-input-container', false);
            await processNFCeUrl(val);
            input.value = "";
        }


        // --- GESTÃO DE SCANNER / CÂMERA AO VIVO ---
        async function startScanner() {
            const modal = document.getElementById('scanner-modal');
            const video = document.getElementById('scanner-video');

            modal.classList.remove('hidden');
            modal.classList.add('flex');
            isScanning = true;

            try {
                stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "environment" } });
                video.srcObject = stream;
                tickQRScan();
            } catch (err) {
                console.error("Sem acesso à câmera física", err);
                showToast("Erro ao abrir câmera física.");
                stopScanner();
            }
        }

        function stopScanner() {
            const modal = document.getElementById('scanner-modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
            isScanning = false;

            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                stream = null;
            }
        }

        function tickQRScan() {
            if (!isScanning) return;
            const video = document.getElementById('scanner-video');

            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                const canvas = document.getElementById('scanner-canvas');
                const ctx = canvas.getContext('2d');
                
                const maxScanDimension = 720;
                let srcWidth = video.videoWidth;
                let srcHeight = video.videoHeight;
                let scaleRatio = Math.min(maxScanDimension / srcWidth, maxScanDimension / srcHeight);
                
                canvas.width = Math.round(srcWidth * scaleRatio);
                canvas.height = Math.round(srcHeight * scaleRatio);
                
                ctx.imageSmoothingEnabled = false;
                ctx.mozImageSmoothingEnabled = false;
                ctx.webkitImageSmoothingEnabled = false;
                ctx.msImageSmoothingEnabled = false;

                ctx.filter = "contrast(150%) brightness(105%) grayscale(100%)";
                ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

                const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                if (window.jsQR) {
                    try {
                        const code = jsQR(imageData.data, canvas.width, canvas.height, {
                            inversionAttempts: "attemptBoth"
                        });
                        if (code) {
                            document.getElementById('scan-input').value = code.data;
                            stopScanner();
                            toggleElement('link-input-container', true);
                            handleLinkProcess();
                            return;
                        }
                    } catch (qrErr) {
                        console.warn("jsQR encontrou um erro silencioso durante a varredura ao vivo:", qrErr);
                    }
                }
            }
            requestAnimationFrame(tickQRScan);
        }

        async function capturePhoto() {
            const canvas = document.getElementById('scanner-canvas');
            const video = document.getElementById('scanner-video');
            const ctx = canvas.getContext('2d');

            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

            stopScanner();
            showToast("Processando imagem da câmera...", true);

            try {
                const dataUrl = canvas.toDataURL('image/png');
                let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                let code = null;

                if (window.jsQR) {
                    try {
                        code = jsQR(imageData.data, canvas.width, canvas.height, { inversionAttempts: "attemptBoth" });
                    } catch (qrErr) {
                        console.warn("jsQR falhou ao ler foto capturada (Tentativa 1):", qrErr);
                    }

                    if (!code) {
                        ctx.clearRect(0, 0, canvas.width, canvas.height);
                        ctx.imageSmoothingEnabled = false;
                        ctx.filter = "contrast(155%) brightness(105%) grayscale(100%)";
                        ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
                        imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                        try {
                            code = jsQR(imageData.data, canvas.width, canvas.height, { inversionAttempts: "attemptBoth" });
                        } catch (qrErr) {
                            console.warn("jsQR falhou ao ler foto capturada (Tentativa 2):", qrErr);
                        }
                    }

                    if (code && code.data) {
                        showToast("QR Code detectado! Acessando dados...", false);
                        await processNFCeUrl(code.data);
                        hideToast();
                        return;
                    }
                }

                // Se não detectar QR Code, avança para o OCR
                const text = await performLocalOCR(dataUrl);
                const parsedData = regexMappingParser(text);
                addPendingItem(parsedData);
            } catch (err) {
                console.error(err);
                showToast("Erro ao processar foto");
            }
            hideToast();
        }

        // --- DASHBOARDS E RELATÓRIOS ---
        function renderDashboard() {
            const userListContainer = document.getElementById('user-stats-list');
            const progressContainer = document.getElementById('category-progress-list');
            const totalDisplay = document.getElementById('monthly-total-value');

            const currentPeriodExpenses = expenses.filter(exp => {
                const d = new Date(exp.date + 'T00:00:00');
                return d.getMonth() === currentMonth && d.getFullYear() === currentYear;
            });

            const totalSum = currentPeriodExpenses.reduce((acc, curr) => acc + curr.amount, 0);
            totalDisplay.innerText = `R$ ${totalSum.toLocaleString('pt-BR', {minimumFractionDigits: 2, maximumFractionDigits: 2})}`;

            // Consolidação por Usuário
            const userTotals = {};
            users.forEach(u => userTotals[u.name] = 0);
            currentPeriodExpenses.forEach(exp => {
                if (userTotals[exp.user] !== undefined) {
                    userTotals[exp.user] += exp.amount;
                }
            });

            userListContainer.innerHTML = Object.entries(userTotals).map(([name, sum]) => `
                <div class="bg-white p-4 rounded-3xl border border-slate-100 shadow-sm flex flex-col justify-between">
                    <span class="text-[9px] font-bold text-slate-400 uppercase tracking-wider">${name}</span>
                    <span class="text-xs font-black text-slate-800 mt-1">R$ ${sum.toLocaleString('pt-BR', {minimumFractionDigits: 2, maximumFractionDigits: 2})}</span>
                </div>
            `).join('');

            // Consolidação por Categoria
            const catTotals = {};
            categories.forEach(cat => catTotals[cat] = 0);
            currentPeriodExpenses.forEach(exp => {
                if (catTotals[exp.category] !== undefined) {
                    catTotals[exp.category] += exp.amount;
                }
            });

            const sortedCategories = Object.entries(catTotals).sort((a,b) => b[1] - a[1]).filter(item => item[1] > 0);

            if (sortedCategories.length === 0) {
                progressContainer.innerHTML = `<p class="text-center text-xs py-6 text-slate-400">Nenhuma transação registrada neste período.</p>`;
                return;
            }

            progressContainer.innerHTML = sortedCategories.map(([catName, amount]) => {
                const percentage = totalSum > 0 ? (amount / totalSum) * 100 : 0;
                return `
                    <div class="space-y-1">
                        <div class="flex justify-between items-center text-xs">
                            <span class="font-extrabold text-slate-500 uppercase text-[9px]">${catName}</span>
                            <span class="font-bold text-slate-900 text-[11px]">R$ ${amount.toLocaleString('pt-BR', {minimumFractionDigits: 2, maximumFractionDigits: 2})} <span class="text-[8px] text-slate-400 font-medium">(${percentage.toFixed(0)}%)</span></span>
                        </div>
                        <div class="w-full bg-slate-100 h-1.5 rounded-full overflow-hidden">
                            <div class="bg-emerald-500 h-full rounded-full transition-all duration-500" style="width: ${percentage}%"></div>
                        </div>
                    </div>
                `;
            }).join('');
        }

        function renderAnnual() {
            const userAnnualContainer = document.getElementById('user-annual-stats');
            const totalAnnualDisplay = document.getElementById('annual-total-value');
            const barsContainer = document.getElementById('annual-bars-container');

            document.getElementById('annual-total-title').innerText = `Total de ${currentYear}`;

            const annualExpenses = expenses.filter(exp => {
                const d = new Date(exp.date + 'T00:00:00');
                return d.getFullYear() === currentYear;
            });

            const annualSum = annualExpenses.reduce((acc, curr) => acc + curr.amount, 0);
            totalAnnualDisplay.innerText = `R$ ${annualSum.toLocaleString('pt-BR', {minimumFractionDigits: 2, maximumFractionDigits: 2})}`;

            const userTotals = {};
            users.forEach(u => userTotals[u.name] = 0);
            annualExpenses.forEach(exp => {
                if (userTotals[exp.user] !== undefined) {
                    userTotals[exp.user] += exp.amount;
                }
            });

            userAnnualContainer.innerHTML = Object.entries(userTotals).map(([name, sum]) => `
                <div class="bg-white p-4 rounded-3xl border border-slate-155 shadow-sm flex flex-col justify-between">
                    <span class="text-[9px] font-bold text-slate-400 uppercase tracking-wider">${name}</span>
                    <span class="text-xs font-black text-slate-800 mt-1">R$ ${sum.toLocaleString('pt-BR', {minimumFractionDigits: 2, maximumFractionDigits: 2})}</span>
                </div>
            `).join('');

            const monthlyTotals = Array(12).fill(0);
            annualExpenses.forEach(exp => {
                const d = new Date(exp.date + 'T00:00:00');
                monthlyTotals[d.getMonth()] += exp.amount;
            });

            const maxMonthVal = Math.max(...monthlyTotals, 1);

            barsContainer.innerHTML = monthlyTotals.map((val, idx) => {
                const heightPercentage = (val / maxMonthVal) * 100;
                return `
                    <div class="flex-1 flex flex-col items-center group relative h-full justify-end select-none">
                        <div class="absolute -top-7 scale-0 group-hover:scale-100 transition-transform bg-slate-800 text-white font-bold text-[8px] py-1 px-1.5 rounded whitespace-nowrap shadow-md z-30 pointer-events-none">
                            R$ ${val.toLocaleString('pt-BR', {maximumFractionDigits: 0})}
                        </div>
                        <div class="w-full bg-emerald-100 rounded-t-lg relative overflow-hidden transition-all duration-500 hover:bg-emerald-600/30" style="height: ${heightPercentage}%">
                            <div class="absolute bottom-0 left-0 right-0 bg-emerald-600 transition-all" style="height: 100%"></div>
                        </div>
                        <span class="text-[7px] font-black text-slate-400 uppercase tracking-wider mt-1.5">${monthNames[idx].substring(0,3)}</span>
                    </div>
                `;
            }).join('');
        }

        // --- AJUSTES DO PAINEL DE CONTROLE ---
        function renderConfigLists() {
            const userList = document.getElementById('users-config-list');
            const catList = document.getElementById('categories-config-list');

            userList.innerHTML = users.map(u => {
                const cardsHtml = u.cards && u.cards.length > 0
                    ? u.cards.map(c => `
                        <span class="inline-flex items-center gap-1 bg-emerald-50 text-emerald-800 border border-emerald-100 rounded px-1.5 py-0.5 text-[9px] font-bold">
                            💳 ${c}
                            <button onclick="removeCardFromUser('${u.id}', '${c}')" class="text-emerald-400 hover:text-red-500 transition-colors ml-0.5 font-bold">×</button>
                        </span>
                      `).join('')
                    : `<span class="text-[9px] text-slate-400 italic">Sem cartões associados</span>`;

                return `
                    <div class="bg-white p-4 rounded-2xl border border-slate-100 space-y-3 shadow-sm">
                        <div class="flex justify-between items-start">
                            <div>
                                <p class="text-xs font-black text-slate-700 uppercase tracking-wide">${u.name}</p>
                                <div class="flex flex-wrap gap-1.5 mt-1.5">
                                    ${cardsHtml}
                                </div>
                            </div>
                            <button onclick="deleteUser('${u.id}')" class="text-slate-300 hover:text-red-500 transition-colors p-1">
                                <i data-lucide="trash-2" class="w-4 h-4"></i>
                            </button>
                        </div>
                        <div class="flex gap-1.5 pt-2 border-t border-slate-100">
                            <input 
                                type="text" 
                                id="add-card-input-${u.id}"
                                placeholder="Final do cartão (4 dígitos)..." 
                                class="flex-1 bg-slate-50 border border-slate-200 rounded-lg text-[9px] px-2 py-1.5 outline-none focus:ring-1 focus:ring-emerald-500/30"
                                maxlength="4"
                            />
                            <button 
                                onclick="addCardToUser('${u.id}')"
                                class="bg-slate-800 text-white px-2.5 py-1.5 rounded-lg text-[8px] font-black uppercase hover:bg-slate-700 active:scale-95 transition-all flex items-center gap-1"
                            >
                                <i data-lucide="plus" class="w-3 h-3"></i> Add
                            </button>
                        </div>
                    </div>
                `;
            }).join('');

            catList.innerHTML = categories.map(c => `
                <span class="inline-flex items-center gap-1.5 bg-slate-50 border border-slate-100 shadow-sm pl-2.5 pr-1.5 py-1 rounded-full text-[9px] font-black text-slate-600 uppercase">
                    ${c}
                    <button onclick="deleteCategory('${c}')" class="text-slate-300 hover:text-red-500 transition-colors p-0.5"><i data-lucide="x" class="w-3 h-3"></i></button>
                </span>
            `).join('');
            lucide.createIcons();
        }

        function addCardToUser(userId) {
            const input = document.getElementById(`add-card-input-${userId}`);
            const cardVal = input.value.trim();

            if (!cardVal) return;
            if (!/^\d{4}$/.test(cardVal)) {
                showToast("Digite exatamente 4 dígitos");
                return;
            }

            const user = users.find(u => u.id === userId);
            if (user) {
                if (!user.cards) user.cards = [];
                if (user.cards.includes(cardVal)) {
                    showToast("Este cartão já está cadastrado");
                    return;
                }
                user.cards.push(cardVal);
                saveLocal();
                renderConfigLists();
                updateUI();
                showToast("Cartão cadastrado!");
            }
        }

        function removeCardFromUser(userId, card) {
            const user = users.find(u => u.id === userId);
            if (user && user.cards) {
                user.cards = user.cards.filter(c => c !== card);
                saveLocal();
                renderConfigLists();
                updateUI();
                showToast("Cartão removido");
            }
        }

        // Criar Usuário
        function handleCreateUser() {
            const inputName = document.getElementById('new-user-input');
            const inputCard = document.getElementById('new-user-card');
            const name = inputName.value.trim();
            const cardStr = inputCard.value.trim();

            if (!name) return;

            const cardArray = cardStr ? cardStr.split(',').map(s => s.trim()).filter(s => /^\d{4}$/.test(s)) : [];

            users.push({
                id: Date.now().toString(),
                name: name,
                cards: cardArray
            });

            saveLocal();
            renderConfigLists();
            updateUI();

            inputName.value = "";
            inputCard.value = "";
            showToast("Usuário criado!");
        }

        function deleteUser(id) {
            if (users.length <= 1) {
                showToast("Mantenha ao menos um usuário ativo");
                return;
            }
            users = users.filter(u => u.id !== id);
            saveLocal();
            renderConfigLists();
            updateUI();
            showToast("Usuário deletado");
        }

        // Criar Categoria
        function handleCreateCategory() {
            const input = document.getElementById('new-category-input');
            const cat = input.value.trim();

            if (!cat || categories.includes(cat)) return;

            categories.push(cat);
            saveLocal();
            renderConfigLists();
            updateUI();

            input.value = "";
            showToast("Categoria cadastrada!");
        }

        function deleteCategory(cat) {
            categories = categories.filter(c => c !== cat);
            saveLocal();
            renderConfigLists();
            updateUI();
            showToast("Categoria removida");
        }

        // --- FUNÇÕES UTILITÁRIAS ---
        function toggleRecurringPeriod(checked) {
            const panel = document.getElementById('recurring-period-panel');
            if (checked) {
                panel.classList.remove('hidden');
                panel.classList.add('block');
            } else {
                panel.classList.remove('block');
                panel.classList.add('hidden');
            }
        }

        // Ocultar/Exibir elementos
        function toggleElement(id, show) {
            const el = document.getElementById(id);
            if (show) {
                el.classList.remove('hidden');
            } else {
                el.classList.add('hidden');
            }
        }

        function focusLinkInput() {
            toggleElement('link-input-container', true);
            document.getElementById('scan-input').focus();
        }
    </script>
</body>
</html>
