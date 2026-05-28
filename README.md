# 🚀 Aula: React + Django CRUD Completo no VSCode

## 📋 Índice
1. [Pré-requisitos](#pré-requisitos)
2. [Configuração do Backend Django](#configuração-do-backend-django)
3. [Configuração do Frontend React](#configuração-do-frontend-react)
4. [Conectando Front e Back](#conectando-front-e-back)
5. [Testando a Aplicação](#testando-a-aplicação)

---

## ✅ Pré-requisitos

Instale no seu computador:
- **Python** (3.8+): https://www.python.org/
- **Node.js** (14+): https://nodejs.org/
- **VSCode**: https://code.visualstudio.com/

Verifique a instalação no terminal:
```bash
python --version
node --version
npm --version
```

---

## 🔧 PARTE 1: Configuração do Backend Django

### Passo 1: Criar pasta do projeto

Abra o VSCode e crie uma pasta para o projeto:

```bash
# No terminal do VSCode (Ctrl + ')
mkdir projeto-crud
cd projeto-crud
```

### Passo 2: Criar ambiente virtual Python (opcional)

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Mac/Linux
python3 -m venv venv
source venv/bin/activate
```

Seu terminal deve mostrar `(venv)` no início.

### Passo 3: Instalar Django

```bash
pip install django djangorestframework django-cors-headers
```

### Passo 4: Criar projeto Django

```bash
django-admin startproject backend .
django-admin startapp tarefas
```

Sua estrutura ficará assim:
```
projeto-crud/
├── venv/
├── backend/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── tarefas/
│   ├── migrations/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   ├── urls.py
│   └── views.py
└── manage.py
```

### Passo 5: Criar o Modelo

Abra `tarefas/models.py` e adicione:

```python
from django.db import models

class Tarefa(models.Model):
    titulo = models.CharField(max_length=200)
    descricao = models.TextField(blank=True)
    concluida = models.BooleanField(default=False)
    data_criacao = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.titulo
    
    class Meta:
        ordering = ['-data_criacao']
```

### Passo 6: Criar Serializer

Crie arquivo `tarefas/serializers.py`:

```python
from rest_framework import serializers
from .models import Tarefa

class TarefaSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tarefa
        fields = ['id', 'titulo', 'descricao', 'concluida', 'data_criacao']
```

### Passo 7: Criar as Views (API)

Abra `tarefas/views.py` e adicione:

```python
from rest_framework import viewsets
from rest_framework.response import Response
from rest_framework.decorators import action
from .models import Tarefa
from .serializers import TarefaSerializer

class TarefaViewSet(viewsets.ModelViewSet):
    queryset = Tarefa.objects.all()
    serializer_class = TarefaSerializer
    
    @action(detail=False, methods=['get'])
    def concluidas(self, request):
        tarefas = Tarefa.objects.filter(concluida=True)
        serializer = self.get_serializer(tarefas, many=True)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'])
    def pendentes(self, request):
        tarefas = Tarefa.objects.filter(concluida=False)
        serializer = self.get_serializer(tarefas, many=True)
        return Response(serializer.data)
```

### Passo 8: Configurar URLs

Crie arquivo `tarefas/urls.py`:

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import TarefaViewSet

router = DefaultRouter()
router.register(r'tarefas', TarefaViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

### Passo 9: Configurar Settings

Abra `backend/settings.py` e faça estas alterações:

**1. Encontre INSTALLED_APPS e adicione:**
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    'rest_framework',
    'corsheaders',
    'tarefas',
]
```

**2. Encontre MIDDLEWARE e adicione no topo:**
```python
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',  # ← Adicione aqui
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

**3. Adicione no final do arquivo:**
```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://127.0.0.1:3000",
]

REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```

### Passo 10: URLs principais

Abra `backend/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('tarefas.urls')),
]
```

### Passo 11: Migração do banco de dados

```bash
python manage.py makemigrations
python manage.py migrate
```

### Passo 12: Criar super usuário (opcional)

```bash
python manage.py createsuperuser
```

### Passo 13: Rodar o servidor Django

```bash
python manage.py runserver
```

O servidor estará em: **http://localhost:8000/api/tarefas/**

---

## ⚛️ PARTE 2: Configuração do Frontend React

### Passo 1: Criar aplicação React

Em um **novo terminal** (sem fechar o Django), navigate até a pasta do projeto:

```bash
cd projeto-crud
npx create-react-app frontend
cd frontend
```

Aguarde a instalação (pode levar alguns minutos).

### Passo 2: Instalar Axios

```bash
npm install axios
```

### Passo 3: Criar arquivo de configuração da API

Crie `frontend/src/api.js`:

```javascript
import axios from 'axios';

const API_BASE_URL = 'http://localhost:8000/api';

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

export default api;
```

### Passo 4: Criar componente de Tarefas

Crie `frontend/src/components/Tarefas.js`:

```javascript
import React, { useState, useEffect } from 'react';
import api from '../api';
import './Tarefas.css';

function Tarefas() {
  const [tarefas, setTarefas] = useState([]);
  const [titulo, setTitulo] = useState('');
  const [descricao, setDescricao] = useState('');
  const [loading, setLoading] = useState(false);

  // Buscar tarefas
  useEffect(() => {
    carregarTarefas();
  }, []);

  const carregarTarefas = async () => {
    try {
      setLoading(true);
      const response = await api.get('/tarefas/');
      setTarefas(response.data);
    } catch (error) {
      console.error('Erro ao buscar tarefas:', error);
    } finally {
      setLoading(false);
    }
  };

  // Criar tarefa
  const handleCriar = async (e) => {
    e.preventDefault();
    if (!titulo.trim()) {
      alert('Por favor, digite um título');
      return;
    }

    try {
      await api.post('/tarefas/', {
        titulo,
        descricao,
        concluida: false,
      });
      setTitulo('');
      setDescricao('');
      carregarTarefas();
    } catch (error) {
      console.error('Erro ao criar tarefa:', error);
    }
  };

  // Atualizar tarefa
  const handleAtualizar = async (id, tarefa) => {
    try {
      await api.put(`/tarefas/${id}/`, {
        ...tarefa,
        concluida: !tarefa.concluida,
      });
      carregarTarefas();
    } catch (error) {
      console.error('Erro ao atualizar tarefa:', error);
    }
  };

  // Deletar tarefa
  const handleDeletar = async (id) => {
    if (window.confirm('Tem certeza que deseja deletar?')) {
      try {
        await api.delete(`/tarefas/${id}/`);
        carregarTarefas();
      } catch (error) {
        console.error('Erro ao deletar tarefa:', error);
      }
    }
  };

  return (
    <div className="container">
      <h1>📝 Minhas Tarefas</h1>

      {/* Formulário */}
      <form onSubmit={handleCriar} className="form">
        <input
          type="text"
          placeholder="Digite o título da tarefa"
          value={titulo}
          onChange={(e) => setTitulo(e.target.value)}
          className="input"
        />
        <textarea
          placeholder="Descrição (opcional)"
          value={descricao}
          onChange={(e) => setDescricao(e.target.value)}
          className="textarea"
        />
        <button type="submit" className="btn-criar">
          ➕ Criar Tarefa
        </button>
      </form>

      {/* Lista de tarefas */}
      {loading ? (
        <p className="loading">Carregando...</p>
      ) : tarefas.length === 0 ? (
        <p className="vazio">Nenhuma tarefa ainda. Crie uma! 🎯</p>
      ) : (
        <ul className="lista">
          {tarefas.map((tarefa) => (
            <li key={tarefa.id} className={tarefa.concluida ? 'concluida' : ''}>
              <div className="tarefa-info">
                <input
                  type="checkbox"
                  checked={tarefa.concluida}
                  onChange={() => handleAtualizar(tarefa.id, tarefa)}
                  className="checkbox"
                />
                <div className="texto">
                  <h3>{tarefa.titulo}</h3>
                  {tarefa.descricao && <p>{tarefa.descricao}</p>}
                  <small>
                    {new Date(tarefa.data_criacao).toLocaleDateString('pt-BR')}
                  </small>
                </div>
              </div>
              <button
                onClick={() => handleDeletar(tarefa.id)}
                className="btn-deletar"
              >
                🗑️
              </button>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default Tarefas;
```

### Passo 5: Criar estilos

Crie `frontend/src/components/Tarefas.css`:

```css
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

h1 {
  color: #333;
  text-align: center;
  margin-bottom: 30px;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 12px;
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 30px;
}

.input,
.textarea {
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 16px;
  font-family: inherit;
}

.textarea {
  resize: vertical;
  min-height: 80px;
}

.input:focus,
.textarea:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 5px rgba(76, 175, 80, 0.3);
}

.btn-criar {
  padding: 12px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-criar:hover {
  background: #45a049;
}

.loading,
.vazio {
  text-align: center;
  color: #666;
  font-size: 18px;
  margin: 30px 0;
}

.lista {
  list-style: none;
  padding: 0;
}

li {
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.3s;
}

li:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

li.concluida {
  background: #f0f0f0;
  opacity: 0.7;
}

li.concluida .texto h3 {
  text-decoration: line-through;
  color: #999;
}

.tarefa-info {
  display: flex;
  gap: 15px;
  flex: 1;
}

.checkbox {
  width: 20px;
  height: 20px;
  cursor: pointer;
  margin-top: 3px;
  flex-shrink: 0;
}

.texto h3 {
  margin: 0 0 5px 0;
  color: #333;
}

.texto p {
  margin: 5px 0;
  color: #666;
  font-size: 14px;
}

.texto small {
  color: #999;
  font-size: 12px;
}

.btn-deletar {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  padding: 5px 10px;
  transition: transform 0.2s;
}

.btn-deletar:hover {
  transform: scale(1.2);
}
```

### Passo 6: Atualizar App.js

Abra `frontend/src/App.js` e substitua o conteúdo:

```javascript
import './App.css';
import Tarefas from './components/Tarefas';

function App() {
  return (
    <div className="App">
      <Tarefas />
    </div>
  );
}

export default App;
```

### Passo 7: Atualizar App.css

Abra `frontend/src/App.css` e substitua:

```css
.App {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  padding: 20px 0;
}

body {
  margin: 0;
  padding: 0;
}
```

### Passo 8: Rodar o React

No terminal da pasta `frontend`:

```bash
npm start
```

O React abrirá em: **http://localhost:3000**

---

## 🔗 PARTE 3: Conectando Front e Back

Para que tudo funcione, você precisa de **dois terminais abertos**:

### Terminal 1 - Django (Backend)
```bash
# Na pasta projeto-crud
source venv/bin/activate  # ou venv\Scripts\activate no Windows
python manage.py runserver
# Rodando em: http://localhost:8000
```

### Terminal 2 - React (Frontend)
```bash
# Na pasta projeto-crud/frontend
npm start
# Rodando em: http://localhost:3000
```

---

## ✅ Testando a Aplicação

### Teste 1: Criar Tarefa
1. Digite um título: "Estudar React"
2. Digite uma descrição: "Aprender hooks"
3. Clique em "➕ Criar Tarefa"
4. A tarefa deve aparecer na lista

### Teste 2: Marcar como Concluída
1. Clique no checkbox de uma tarefa
2. O item deve ficar acinzentado
3. O título deve ficar com risco

### Teste 3: Deletar Tarefa
1. Clique no botão 🗑️
2. Confirme a exclusão
3. A tarefa deve sumir da lista

### Teste 4: Verificar dados no Django Admin
1. Acesse: http://localhost:8000/admin/
2. Use as credenciais criadas no `createsuperuser`
3. Veja todas as tarefas criadas

---

## 🎯 Resumo do CRUD

| Operação | Método | URL | Função |
|----------|--------|-----|--------|
| **C**reate | POST | `/api/tarefas/` | Criar tarefa |
| **R**ead | GET | `/api/tarefas/` | Listar tarefas |
| **U**pdate | PUT | `/api/tarefas/{id}/` | Atualizar tarefa |
| **D**elete | DELETE | `/api/tarefas/{id}/` | Deletar tarefa |

---

## 🐛 Erros Comuns

### "CORS error" no console
**Solução:** Verifique se `CORS_ALLOWED_ORIGINS` em `settings.py` tem `http://localhost:3000`

### "Cannot GET /api/tarefas/"
**Solução:** Certifique-se de que as migrations foram rodadas: `python manage.py migrate`

### "Tarefa não aparece na lista"
**Solução:** Abra o console do navegador (F12) e veja se há erros

### Porta 8000 ou 3000 já está em uso
**Solução:**
```bash
# Mudar Django para porta 8001
python manage.py runserver 8001

# Mudar React (editar package.json)
"start": "PORT=3001 react-scripts start"
```

---

## 📚 Próximos Passos

1. **Adicionar autenticação** - Login de usuários
2. **Filtros avançados** - Por data, status, etc
3. **Paginação** - Mostrar 10 tarefas por página
4. **Notificações** - Toast de sucesso/erro
5. **Deploy** - Colocar no Heroku ou Vercel

---

## 📖 Referências

- [Django REST Framework](https://www.django-rest-framework.org/)
- [React Docs](https://react.dev/)
- [Axios Documentation](https://axios-http.com/)
- [Django Documentation](https://docs.djangoproject.com/)

---

**Sucesso! 🎉 Você tem uma aplicação CRUD funcional!**
