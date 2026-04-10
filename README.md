# Aula 10/04/2026 - Gerenciamento de Estado e Persistência de Dados

## Objetivo da Aula

Na aula anterior organizamos nosso projeto separando estilos, lógica e telas, deixando o código mais profissional e escalável.

Hoje vamos evoluir o **comportamento** do aplicativo.

Até agora nosso app funciona, mas tem uma limitação importante:

> → Toda vez que recarregamos, os dados voltam ao estado inicial

Nesta aula vamos resolver isso e introduzir dois pilares fundamentais no desenvolvimento mobile:

> → Gerenciamento de estado com `useState`  
> → Persistência de dados com `AsyncStorage`

---

## O Que Vamos Aprender Hoje

- O que é estado (state) em aplicações React Native
- Como o hook `useState` controla mudanças na interface
- Como funciona persistência local no mobile
- Como salvar e recuperar dados com `AsyncStorage`
- Como integrar estado + persistência no fluxo do app

---

## Por Que Isso É Importante?

| Sem estado e persistência | Com estado + persistência |
|---|---|
| ❌ Interface não reage corretamente às ações do usuário | ✅ Interface dinâmica e reativa |
| ❌ Dados são perdidos ao recarregar | ✅ Dados permanecem após fechar o app |
| ❌ Aplicação não é confiável | ✅ Experiência real de produto |

---

## PARTE 1: Entendendo o Estado (useState)

### O Que é Estado (state) no React?

Estado é qualquer informação que pode mudar ao longo do tempo dentro do aplicativo.

No nosso projeto:

- Status da consulta
- Dados do paciente
- Informações exibidas na tela

### O Hook useState

```typescript
const [valor, setValor] = useState(valorInicial);
```

| Elemento | Interpretação |
|---|---|
| `valor` | dado atual |
| `setValor` | função que atualiza o dado |
| `valorInicial` | estado inicial do componente |

### Exemplo Aplicado no Projeto

> Arquivo: `src/screens/Home.tsx`

```typescript
const [consulta, setConsulta] = useState<Consulta>(consultaInicial);
```

Aqui estamos dizendo:

- O estado será do tipo `Consulta`
- Começa com `consultaInicial`
- Qualquer mudança deve passar por `setConsulta`

### Atualizando Estado (Forma Correta)

```typescript
function confirmarConsulta() {
  const novaConsulta = {
    ...consulta,
    status: "confirmada" as const,
  };
  setConsulta(novaConsulta);
}
```

### Regra de Ouro no React

❌ **Nunca fazer isso:**

```typescript
consulta.status = "confirmada"; // ERRADO
```

✅ **Sempre fazer isso:**

```typescript
setConsulta({ ...consulta, status: "confirmada" }); // CORRETO
```

> **Motivo:** React detecta mudanças por referência. Alteração direta não dispara re-render.

---

## PARTE 2: O Problema da Persistência

**Situação atual:**

```
Usuário confirma consulta
    ↓
Status muda para "confirmada"
    ↓
Atualiza a página
    ↓
Volta para "agendada"  ← problema!
```

**Por quê?**

> O estado fica apenas na memória (RAM). Ao recarregar, tudo é perdido.

---

## PARTE 3: Introdução ao AsyncStorage

### O Que É?

Um armazenamento local do React Native baseado em **chave-valor**.

Funciona assim:

```
→ Você salva com uma chave
→ Recupera usando a mesma chave
```

### Instalando o AsyncStorage

```bash
npm install @react-native-async-storage/async-storage
```

**O que esse comando faz?**

- Utiliza o gerenciador de pacotes `npm`
- Baixa a biblioteca do `AsyncStorage` do repositório oficial
- Adiciona a dependência no arquivo `package.json`
- Instala os arquivos dentro da pasta `node_modules`

**Que biblioteca é essa?**

- `@react-native-async-storage/async-storage`
- Biblioteca oficial para persistência local no React Native
- Funciona como armazenamento chave-valor
- Substitui o antigo `AsyncStorage` que existia dentro do React Native core

> 💡 **Tradução prática:** "Esse comando instala a ferramenta que permite o aplicativo guardar dados no próprio celular ou navegador, sem precisar de backend."

### Importando

```typescript
import AsyncStorage from "@react-native-async-storage/async-storage";
```

### Definindo a Chave

```typescript
const STORAGE_KEY = "@consultas:consulta_atual";
```

> **Boa prática:** use prefixo com `@` e um nome claro e organizado.

---

## PARTE 4: Salvando Dados

```typescript
async function salvarConsulta(consultaAtualizada: Consulta) {
  try {
    await AsyncStorage.setItem(
      STORAGE_KEY,
      JSON.stringify(consultaAtualizada)
    );
  } catch (erro) {
    console.error("Erro ao salvar:", erro);
  }
}
```

**Pontos importantes:**

- `AsyncStorage` só salva `string`
- `JSON.stringify` converte objeto → string
- `async/await` garante controle da execução

---

## PARTE 5: Carregando Dados

```typescript
async function carregarConsulta() {
  try {
    const dados = await AsyncStorage.getItem(STORAGE_KEY);
    if (dados) {
      const consultaObj = JSON.parse(dados);
      consultaObj.data = new Date(consultaObj.data);
      setConsulta(consultaObj);
    }
  } catch (erro) {
    console.error("Erro ao carregar:", erro);
  }
}
```

---

## PARTE 6: Executando no Carregamento (useEffect)

```typescript
useEffect(() => {
  carregarConsulta();
}, []);
```

> **Interpretação:** executa apenas uma vez, ideal para carregar dados iniciais.

---

## PARTE 7: Integração Completa

Agora conectamos tudo:

- Estado (`useState`)
- Persistência (`AsyncStorage`)
- Ciclo de vida (`useEffect`)

### Código Atualizado da Tela

> Arquivo: `src/screens/Home.tsx`

```tsx
import React, { useState, useEffect } from "react";
import { View, Text, ScrollView } from "react-native";
import { StatusBar } from "expo-status-bar";
import AsyncStorage from "@react-native-async-storage/async-storage";
import { Especialidade } from "../types/especialidade";
import { Paciente } from "../types/paciente";
import { Medico } from "../interfaces/medico";
import { Consulta } from "../interfaces/consulta";
import { ConsultaCard } from "../components";
import { styles } from "../styles/app.styles";

const STORAGE_KEY = "@consultas:consulta_atual";

export default function Home() {
  const cardiologia: Especialidade = {
    id: 1,
    nome: "Cardiologia",
    descricao: "Cuidados com o coração",
  };

  const medico1: Medico = {
    id: 1,
    nome: "Dr. Roberto Silva",
    crm: "CRM12345",
    especialidade: cardiologia,
    ativo: true,
  };

  const paciente1: Paciente = {
    id: 1,
    nome: "Carlos Andrade",
    cpf: "123.456.789-00",
    email: "carlos@email.com",
    telefone: "(11) 98765-4321",
  };

  const consultaInicial: Consulta = {
    id: 1,
    medico: medico1,
    paciente: paciente1,
    data: new Date(2026, 2, 10),
    valor: 350,
    status: "agendada",
    observacoes: "Consulta de rotina",
  };

  const [consulta, setConsulta] = useState<Consulta>(consultaInicial);

  useEffect(() => {
    carregarConsulta();
  }, []);

  async function carregarConsulta() {
    try {
      const consultaSalva = await AsyncStorage.getItem(STORAGE_KEY);
      if (consultaSalva) {
        const consultaObjeto = JSON.parse(consultaSalva);
        consultaObjeto.data = new Date(consultaObjeto.data);
        setConsulta(consultaObjeto);
      }
    } catch (erro) {
      console.error("Erro ao carregar consulta:", erro);
    }
  }

  async function salvarConsulta(consultaAtualizada: Consulta) {
    try {
      await AsyncStorage.setItem(
        STORAGE_KEY,
        JSON.stringify(consultaAtualizada)
      );
    } catch (erro) {
      console.error("Erro ao salvar consulta:", erro);
    }
  }

  function confirmarConsulta() {
    const consultaAtualizada = {
      ...consulta,
      status: "confirmada" as const,
    };
    setConsulta(consultaAtualizada);
    salvarConsulta(consultaAtualizada);
  }

  function cancelarConsulta() {
    const consultaAtualizada = {
      ...consulta,
      status: "cancelada" as const,
    };
    setConsulta(consultaAtualizada);
    salvarConsulta(consultaAtualizada);
  }

  return (
    <View style={styles.container}>
      <StatusBar style="light" />
      <ScrollView contentContainerStyle={styles.scrollContent}>
        <View style={styles.header}>
          <Text style={styles.titulo}>Sistema de Consultas</Text>
          <Text style={styles.subtitulo}>Consulta #{consulta.id}</Text>
        </View>
        <ConsultaCard
          consulta={consulta}
          onConfirmar={confirmarConsulta}
          onCancelar={cancelarConsulta}
        />
      </ScrollView>
    </View>
  );
}
```

---

## PARTE 8: Fluxo Completo

```
App inicia
    ↓
useState cria estado inicial
    ↓
useEffect executa → carregarConsulta() tenta recuperar dados
    ↓
Usuário interage → estado muda via setConsulta
    ↓
Dados são persistidos → AsyncStorage salva
    ↓
App recarrega → dados são restaurados ✅
```

---

## PARTE 9: Testes em Sala

### Teste 1

1. Confirmar consulta
2. Recarregar página
3. Status deve **permanecer** como "confirmada"

### Teste 2

1. Cancelar consulta
2. Recarregar
3. Status deve **persistir** como "cancelada"

### Teste 3 — Debug

1. Abrir DevTools (`F12`)
2. Ir em **Application → Local Storage**
3. Localizar a chave `@consultas:consulta_atual`

---

## Resumo da Aula

| Hook / API | Para que serve |
|---|---|
| `useState` | Controla dados dinâmicos e dispara re-render |
| `AsyncStorage` | Salva dados localmente e mantém estado entre sessões |
| `useEffect` | Executa código no ciclo de vida do componente |

---

> Por hoje é isso! 
