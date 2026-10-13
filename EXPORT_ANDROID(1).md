# Exportação completa para desenvolvimento Android

Este documento organiza a entrega do projeto como base para um **APK Android instalável**
(nome próprio, ícone, orientação horizontal travada e integrações nativas).

---

## 1. O que este projeto é

Interface de desktop inspirada no Windows 11, escrita em **React 18 + Vite + Tailwind CSS (JavaScript)**,
com persistência local em `localStorage`. Nenhum recurso exclusivo de um modelo Samsung é usado —
tudo é CSS/JS padrão, testado visando Samsung Galaxy A14 5G (Android 13/14) e tablets Galaxy.

## 2. Como exportar TODOS os arquivos (código-fonte completo)

O projeto inteiro já está em código-fonte versionável. Duas formas:

**A. Sincronia Git (recomendada)** — no painel do Base44, conecte sua conta GitHub
(ícone GitHub / integrações). O repositório recebe **todos** os arquivos abaixo,
preservando caminhos e referências:

```
index.html                  ← HTML raiz (viewport-fit=cover p/ áreas seguras)
package.json                ← dependências e scripts de construção
package-lock.json           ← bloqueio de dependências
vite.config.js              ← configuração do framework (Vite + plugin Base44)
tailwind.config.js          ← configuração do Tailwind (tokens, cores, fontes)
postcss.config.js           ← PostCSS
components.json             ← configuração shadcn/ui
jsconfig.json               ← alias @/ e escopo do projeto
eslint.config.js
src/
  main.jsx                  ← entrada do React
  App.jsx                   ← roteador (rota "/": Desktop)
  index.css                 ← tokens de design (cores, fontes)
  api/base44Client.js       ← cliente do serviço de backend
  lib/                      ← desktopStore, vfs, useClock, utils, AuthContext…
  pages/Desktop.jsx         ← orquestrador do desktop
  components/desktop/       ← Wallpaper, DesktopIcons, Taskbar, StartMenu, PowerMenu, RotatePrompt
  components/windows/       ← Window (arrastar/redimensionar)
  components/ui/            ← componentes shadcn
  apps/                     ← registry + os 9 aplicativos
```

**B. Build estático** — `npm install && npm run build` gera `dist/` com HTML/CSS/JS finais,
pronto para servir dentro de um WebView.

## 3. Instalar, executar, testar e compilar (web)

```bash
npm install          # instala as dependências da lista abaixo
npm run dev          # servidor de desenvolvimento
npm run build        # compila para dist/
npm run lint         # verificação de código
```

## 4. Dependências e versões (package.json)

Runtime: react 18.2, react-dom 18.2, react-router-dom 6.26, @tanstack/react-query 5.84,
@base44/sdk 0.8.53, tailwindcss 3.x, lucide-react 0.475, framer-motion 11.16, recharts 2.15,
date-fns 3.6, lodash 4.17, react-hook-form 7.54, zod 3.24, class-variance-authority 0.7,
clsx 2.1, tailwind-merge 3.0, tailwindcss-animate 1.0, radix-ui (accordion, dialog, dropdown-menu,
popover, select, switch, tabs, tooltip etc. 1.x), cmdk 1.0, sonner 2.0, three 0.171.
Dev: vite 5.x, @vitejs/plugin-react, typescript 5.x, eslint 8/9, postcss 8.x, autoprefixer 10.x.
A lista exata e travada está em `package.json` + `package-lock.json` no repositório.

**Variáveis de ambiente / integrações:** o app não exige chaves secretas do usuário.
O bootstrap autentica no serviço de nuvem do Base44 (definido em `src/lib/app-params.js`).
Não há senhas nem credenciais privadas no código.

## 5. Recursos WEB vs. recursos que exigem implementação NATIVA Android

### Já funciona 100% na web (web + WebView, sem permissões)
Área de trabalho, papel de parede, barra de tarefas translúcida, relógio real (fuso configurável,
`America/Sao_Paulo` padrão ou fuso do dispositivo), menu Iniciar com pesquisa, janelas arrastáveis/
redimensionáveis com clamp na tela, Calculadora, Bloco de Notas (localStorage), Calendário com
eventos (localStorage), Explorador de Arquivos virtual + Lixeira, Navegador (iframe com fallback
"abrir em nova aba"), Configurações persistidas, Gerenciador de Aplicativos, Este Computador,
aviso de rotação em retrato, áreas seguras (`env(safe-area-inset-*)`), rolagem interna nas janelas.

### Exige código nativo Android (não simulado — não implementado)
| Função | Como implementar |
|---|---|
| Travamento de orientação horizontal | `android:screenOrientation="sensorLandscape"` no `AndroidManifest.xml` (o aviso web some sozinho em toque + retrato) |
| Desligar/reiniciar o APARELHO | Impossível por app comum; requer Device Owner/Admin ou não implementar |
| Arquivos físicos do dispositivo | Storage Access Framework (SAF) ou `MANAGE_EXTERNAL_STORAGE` + bridge ao `src/lib/vfs.js` |
| Status real de Wi-Fi/bateria | `ConnectivityManager`/`BatteryManager` via ponte JS (Capacitor plugin) |
| Volume real do sistema | `AudioManager` via plugin nativo |
| Notificações push | Firebase Cloud Messaging |
| Instalação de apps reais | `Intent` com `ACTION_INSTALL_PACKAGE` (requer permissão do usuário) |

### Dependência de decisão
O bootstrap do app autentica no serviço Base44 (exige internet no APK). A interface do desktop
em si é offline (localStorage). Para um APK totalmente offline, substitua o `AuthProvider`
(`src/lib/AuthContext.jsx`) por login local ou carregue a URL publicada no WebView.

## 6. Receita de APK com Capacitor (recomendada)

```bash
# dentro do repositório exportado
npm install @capacitor/core @capacitor/cli @capacitor/android
npx cap init "Desktop Samsung" com.exemplo.desktop --web-dir=dist
npm run build
npx cap add android
```

Edite `android/app/src/main/AndroidManifest.xml` na activity principal:
```xml
android:screenOrientation="sensorLandscape"
```
Ícone: gere em `android/app/src/main/res/mipmap-*` (Image Asset no Android Studio).
Depois:

```bash
npx cap sync
npx cap open android     # abre o Android Studio
# Build > Build Bundle(s)/APK(s) > Build APK(s)
```

Um `capacitor.config.example.json` está incluído neste projeto como referência.

## 7. Status da verificação (item 13)

Testado no preview da plataforma (ver relatório na conversa):
- Botões e atalhos do Iniciar, barra de tarefas, janelas, Calculadora, energia ✓
- Barra inferior e relógio visíveis em telas pequenas ✓ (sem corte, sem overflow horizontal)
- Preferências persistem após recarregar ✓ (localStorage)
- Ícones da área de trabalho reorganizam em colunas em telas baixas ✓
- Janelas abertas/movidas/redimensionadas são presas dentro da área visível ✓
- Proporção exata do Galaxy A14 5G em campo + Android 13/14/15 reais: **testar no dispositivo** —
  não declarar compatibilidade total sem esses testes físicos.