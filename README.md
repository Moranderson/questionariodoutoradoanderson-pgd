# questionariodoutoradoanderson-pgd
import os
import google.auth
from googleapiclient.discovery import build
from google.oauth2.credentials import Credentials
from google.auth.transport.requests import Request
from google_auth_oauthlib.flow import InstalledAppFlow

# Escopos necessários para acessar o Google Forms
SCOPES = ['https://www.googleapis.com/auth/forms.body']

def authenticate_google():
    """Autentica no Google e retorna as credenciais."""
    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file('credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        with open('token.json', 'w') as token:
            token.write(creds.to_json())
    return creds

def create_google_form(service, title, description):
    """Cria um novo formulário no Google Forms."""
    form = {
        "info": {
            "title": title,
            "documentTitle": title,
            "description": description,
        }
    }
    result = service.forms().create(body=form).execute()
    return result['formId']

def add_question_to_form(service, form_id, question):
    """Adiciona uma pergunta ao formulário."""
    requests = [
        {
            "createItem": {
                "item": question,
                "location": {
                    "index": 0
                }
            }
        }
    ]
    service.forms().batchUpdate(formId=form_id, body={'requests': requests}).execute()

def main():
    # Autenticar no Google
    creds = authenticate_google()
    service = build('forms', 'v1', credentials=creds)

    # Criar um novo formulário
    form_id = create_google_form(service, "Questionário PGDP na Saúde", "Este questionário avalia a implantação e utilidade do PGDP na saúde.")

    # Adicionar perguntas
    questions = [
        {
            "title": "Qual é a sua área de atuação na saúde?",
            "questionItem": {
                "question": {
                    "required": True,
                    "choiceQuestion": {
                        "type": "RADIO",
                        "options": [
                            {"value": "Pesquisa Clínica"},
                            {"value": "Epidemiologia"},
                            {"value": "Saúde Pública"},
                            {"value": "Bioinformática"},
                            {"value": "Outro"}
                        ]
                    }
                }
            }
        },
        {
            "title": "Qual é o seu nível de experiência com PGDP?",
            "questionItem": {
                "question": {
                    "required": True,
                    "choiceQuestion": {
                        "type": "RADIO",
                        "options": [
                            {"value": "Iniciante"},
                            {"value": "Intermediário"},
                            {"value": "Avançado"},
                            {"value": "Nenhuma experiência"}
                        ]
                    }
                }
            }
        },
        # Adicione mais perguntas aqui
    ]

    for question in questions:
        add_question_to_form(service, form_id, question)

    print(f"Formulário criado com sucesso! Acesse em: https://docs.google.com/forms/d/{form_id}/edit")

if __name__ == "__main__":
    main()
