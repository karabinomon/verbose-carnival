# Guia: Como criar e rodar um PWA local com Vite, Vue, Axios e BrasilAPI

Este guia ensina a configurar um Progressive Web App (PWA) em ambiente de desenvolvimento local usando **Vite**, **Vue 3**, **Axios** e a biblioteca **Vite PWA Plugin**.

---

### Passo 1: Criar o Projeto Vite + Vue
No terminal, crie um novo projeto usando o Vite:
```bash
npm create vite@latest meu-pwa-brasilapi -- --template vue
cd meu-pwa-brasilapi
npm install
```

### Passo 2: Instalar o Axios e o Plugin PWA do Vite
Instale o `axios` para as requisições HTTP e o `vite-plugin-pwa` para a configuração do Service Worker e Manifesto.
```bash
npm install axios
npm install -D vite-plugin-pwa
```

### Passo 3: Configurar o `vite.config.js`
Abra o arquivo `vite.config.js` e configure o plugin do PWA. É importante habilitar `devOptions.enabled: true` para que o Service Worker funcione e possa ser testado no ambiente de desenvolvimento local (`npm run dev`).

Substitua o conteúdo de `vite.config.js` por:
```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig({
  plugins: [
    vue(),
    VitePWA({
      registerType: 'autoUpdate',
      devOptions: {
        enabled: true // Habilita o PWA no modo de desenvolvimento local
      },
      manifest: {
        name: 'Busca Banco BrasilAPI',
        short_name: 'BuscaBanco',
        description: 'Buscar informações de bancos usando a BrasilAPI',
        theme_color: '#ffffff',
        icons: [
          {
            src: 'pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png'
          },
          {
            src: 'pwa-512x512.png',
            sizes: '512x512',
            type: 'image/png'
          }
        ]
      }
    })
  ]
})
```

### Passo 4: Criar a funcionalidade de busca no `src/App.vue`
Substitua o conteúdo de `src/App.vue` para conter a caixa de busca, o botão e a integração com a BrasilAPI via Axios. Não há estilização ou CSS.

```html
<script setup>
import { ref } from 'vue'
import axios from 'axios'

const codigoBanco = ref('')
const banco = ref(null)
const erro = ref('')

async function buscarBanco() {
  banco.value = null
  erro.value = ''
  
  if (!codigoBanco.value) {
    erro.value = 'Por favor, digite o código do banco.'
    return
  }

  try {
    const response = await axios.get(`https://brasilapi.com.br/api/banks/v1/${codigoBanco.value}`)
    banco.value = response.data
  } catch (err) {
    erro.value = 'Banco não encontrado ou erro na requisição.'
  }
}
</script>

<template>
  <div>
    <h1>Busca de Bancos - BrasilAPI</h1>
    
    <!-- Caixa de busca e botão -->
    <input 
      v-model="codigoBanco" 
      type="text" 
      placeholder="Digite o código do banco (ex: 1)" 
    />
    <button @click="buscarBanco">Buscar</button>

    <!-- Exibição do resultado -->
    <div v-if="banco">
      <h2>Resultado:</h2>
      <p><strong>Nome:</strong> {{ banco.name }}</p>
      <p><strong>Nome Completo:</strong> {{ banco.fullName }}</p>
      <p><strong>Código:</strong> {{ banco.code }}</p>
      <p><strong>ISPB:</strong> {{ banco.ispb }}</p>
    </div>

    <!-- Mensagem de erro -->
    <div v-if="erro">
      <p style="color: red;">{{ erro }}</p>
    </div>
  </div>
</template>
```

### Passo 5: Como rodar e testar o PWA localmente

Existem duas formas principais de rodar o projeto localmente para validar o Service Worker e o manifesto PWA:

#### Opção A: Executar em Modo de Desenvolvimento
Execute o servidor de desenvolvimento:
```bash
npm run dev
```
1. Abra o navegador no endereço indicado (geralmente `http://localhost:5173`).
2. Abra as Ferramentas do Desenvolvedor (F12) -> aba **Application** (ou **Aplicativo**).
3. No menu lateral, clique em **Service Workers** para verificar se ele está registrado e ativo.
4. Em **Manifest**, você poderá validar o nome e ícones configurados.

#### Opção B: Buildar e Visualizar a Versão de Produção (Recomendado para testar offline completo)
Para simular com precisão o comportamento real do PWA:
1. Faça o build do projeto:
   ```bash
   npm run build
   ```
2. Inicie o servidor de pré-visualização local:
   ```bash
   npm run preview
   ```
3. Acesse a URL gerada (geralmente `http://localhost:4173`).
4. O navegador mostrará o ícone de instalação do PWA diretamente na barra de endereços.
