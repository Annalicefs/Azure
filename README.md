# Tradução de Artigos com Extração de Texto e Integração com APIs

Este projeto realiza a extração de texto de páginas web e traduz o conteúdo utilizando APIs como o **Microsoft Translator** e o **Azure OpenAI**.

---

## Requisitos

- Python 3.8+
- Bibliotecas necessárias:  
  ```bash
  pip install requests python-docx beautifulsoup4 openai langchain-openai
  ```
---

### 1. Tradução de Texto com Microsoft Translator

A API **Microsoft Translator** é utilizada para traduzir strings simples.  

#### Exemplo:
```python
import requests
from docx import Document
import os

subscription_key = "API-KEY"
endpoint = 'ENDPOINT'
location = "LOCAL"
target_language = 'pt-br'

def translator_text(text, target_language):
  path = '/translate'
  constructed_url = endpoint + path
  headers = {
      'Ocp-Apim-Subscription-Key': subscription_key,
      'Ocp-Apim-Subscription-Region': location,
      'Content-type': 'application/json',
      'X-ClientTraceId': str(os.urandom(16))
  }

  body = [{
      'text': text
  }]

  params = {
      'api-version': '3.0',
      'from': 'en',
      'to': target_language
  }

  request = requests.post(constructed_url, params=params, headers=headers, json=body)
  response = request.json()
  return response[0]["translations"][0]["text"]
```

---

### 2. Extração de Texto de uma URL

A extração de texto é feita através do módulo **BeautifulSoup**, que remove scripts e estilos do HTML.  

#### Exemplo:
```python
import requests
from bs4 import BeautifulSoup

def extraindo_texto_url(url):
    response = requests.get(url)

    if response.status_code == 200:
      soup = BeautifulSoup(response.text, 'html.parser')

      for script_or_style in soup(["script", "style"]):
        script_or_style.decompose()

      texto = soup.get_text(separator = '')

      #Limpando texto
      linhas = (line.strip() for line in texto.splitlines())
      parte_texto = (phrase.strip() for line in linhas for phrase in line.split("  "))
      texto = '\n'.join(chunk for chunk in parte_texto if chunk)
      return texto
    else:
      raise Exception("Falha ao acessar a URL")

extraindo_texto_url('URL')
```

---

### 3. Integração com Azure OpenAI

A integração utiliza o modelo **GPT-4o-mini** para tradução. .

#### Exemplo:
```python
from langchain_openai.chat_models.azure import AzureChatOpenAI

client = AzureChatOpenAI(
    azure_endpoint = "ENDPOINT",
    api_key = "API-KEY",
    api_version = "VERSÃO",
    deployment_name = "MODELO",
    max_retries = 0
)

def translate_article(text, lang):
  messages = [
    ("system" , "Você atua como tradutor de textos"),
    ("user", f"Traduza o {text} para o idioma {lang} e responda em markdown")
  ]

  response = client.invoke(messages)
  print(response.content)
  return response.content

translate_article("TEXTO", "portugues")

```

```python
url = 'URL'
texto = extraindo_texto_url(url)
artigo = texto_traduzido = translate_article(texto, "portugues")
print(artigo)

```



