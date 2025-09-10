# Warsztat: Wirtualny Doradca Ubezpieczeniowy z Obsługą Wielokanałową

Celem warsztatu jest stworzenie interfejsu API do integracji z chatbotem generatywnym, który może odpowiadać na pytania klientów dotyczące polis ubezpieczeniowych, przetwarzając dane w czasie rzeczywistym i korzystając z Azure API Management do zarządzania, monitorowania i zabezpieczania API.

## Wymagania dla uczestników

Przed przystąpieniem do warsztatu, upewnij się że posiadasz:

- Aktywną subskrypcję Azure (lub darmowe środki)
- Wdrożoną usługę Azure Open AI z dostępnym modelem np. "gpt-4o-mini"
https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/create-resource?pivots=web-portal
- Wdrożoną usługe "Azure Log Analytics"
https://learn.microsoft.com/en-us/azure/api-management/monitor-api-management
- Wdrożoną usługe "Application Insight"
https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-app-insights?tabs=rest
- Zainstalowane narzędzia:
    - Azure CLI (wersja 2.40.0 lub wyższa)
    - Visual Studio Code lub inne IDE
    - Postman lub inny klient REST (opcjonalnie)
- Podstawową znajomość REST API: żądania HTTP, nagłówki, kody odpowiedzi

---

## 1. TWORZENIE PIERWSZEGO API (BAZA WIEDZY O POLISACH)

### 1.1 Przygotowanie środowiska

1. Zaloguj się do Azure Portal (portal.azure.com)  
2. Sprawdź czy posiadasz aktywną subskrypcję

### 1.2 Tworzenie usługi Azure API Management

https://learn.microsoft.com/en-us/azure/api-management/get-started-create-service-instance

1. W Azure Portal, wyszukaj "API Management" w pasku wyszukiwania  
2. Kliknij "+ Create" lub "Utwórz"  
3. Wypełnij formularz:
    - **Subscription**: wybierz swoją subskrypcję
    - **Resource Group**: utwórz nową (np. "rg-workshop-apim")
    - **Region**: wybierz najbliższy (np. France Central)
    - **Name**: np. "workshop-apim"
    - **Organization name**: nazwa Twojej organizacji
    - **Administrator email**: Twój adres email
    - **Pricing tier**: Developer (najtańsza, nieprodukcyjna opcja)
4. W zakładce "Monitor + Secure", zaznacz opcje "Log Analytics" oraz "Application Insights", wybierz wcześniej utworzone zasoby.
5. W zakładce "Virtual Network", zaznacz opcję "Virtual Network" a następnie z "Type" wybierz "External" poprzez opcję "Create new" utwórz nową sieć wirtualną, wprowadź nazwę i możesz zaakceptować domyślną adresację. 
6. W zakładce "Managed identity" w polu "Status" zaznacz "checkbox"
7. Kliknij "Review + create", a następnie "Create"
8. Poczekaj na zakończenie wdrażania (może potrwać 30-40 minut)

### 1.3 Definiowanie modelu danych dla polis

https://learn.microsoft.com/en-us/azure/api-management/add-api-manually

Dla naszego API będziemy używać następującego modelu danych polisy:

- ID (unikalny identyfikator)
- Rodzaj polisy (np. zdrowotna, samochodowa, mieszkaniowa)
- Dostępne pakiety (np. premium, standard)
- Cena (miesięczna)
- Opis (co polisa obejmuje)

### 1.4 Tworzenie API dla bazy wiedzy o polisach

1. Przejdź do utworzonego zasobu API Management
2. W menu bocznym wybierz "APIs", następnie jeszcze raz APIs.
3. Kliknij "+ Add API" i wybierz "HTTP API"
4. Wypełnij formularz:
    - **Display name**: PolisyAPI
    - **Name**: polisy-api
    - **Web service URL**: można tymczasowo wpisać "https://example.org"
    - **API URL suffix**: polisy
5. Kliknij "Create"

### 1.5 Dodawanie operacji GET /polisy

1. Wybierz utworzone API "PolisyAPI"
2. Kliknij "+ Add operation"
3. Wypełnij formularz:
    - **Display name**: GetPolisy
    - **Name**: getpolisy
    - **URL**: GET /polisy
    - **Description**: Pobiera listę dostępnych polis
4. W sekcji "Responses" kliknij "+ Add response"
    - **Status code**: 200 OK
    - W "Representations" kliknij "Add representation" wybierz "application/json"
    - Wklej przykładowy schemat:

```json
[
  {
    "polisaId": "123456",
    "rodzajPolisy": "zdrowotna",
    "pakiet": "premium",
    "cena": 100,
    "opis": "Ubezpieczenie zdrowotne premium."
  },
  {
    "polisaId": "123457",
    "rodzajPolisy": "samochodowa",
    "pakiet": "standard",
    "cena": 75,
    "opis": "Podstawowe ubezpieczenie samochodu."
  }
]
```

5. Kliknij "Save"
6. Przejdź do zakładki "Settings" W sekcji "Subscription" odznacz opcję "Subscription required" (dla celów testowych)
7. Kliknij "Save"
8. Wybierz utworzone API i przejdź do "Inbound processing"
9. Kliknij "Add policy", wybierz "mock-response"
10. Kliknij "Save"

### 1.6 Testowanie API

https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-api-inspector

1. Wybierz utworzone API i przejdź do zakładki "Test"
2. Wybierz operację GET /polisy
3. Kliknij "Send"
4. Sprawdź czy otrzymujesz odpowiedź z przykładowymi danymi polis

---

## 2. UDOSTĘPNIENIE OPEN AI POPRZEZ APIM

https://learn.microsoft.com/en-us/azure/api-management/azure-openai-api-from-specification

### 2.1 Dodawanie API Azure OpenAI

1. W zasobie API Management przejdź do sekcji "APIs"
2. Kliknij "+ Add API" i wybierz "Azure OpenAI Service"
3. Wypełnij formularz:
    - Wybierz dostępną instancję Azure OpenAI
    - **Display name**: polisy-ai
    - **name**: polisy-ai
    - Podaj dowolny opis
    - Zaznacz opcję "Improve SDK Compatibility"
4. Kliknij "Next"
5. Zaznacz opcję "Track token usage" (potrzebne do rozliczalności)
6. Wybierz dostępną instancję Application Insights jako miejsce do odkładania metryk tokenów
7. W opcji "dimension" wybierz: User ID, API ID
8. Kliknij "Review + create"
9. kliknij "Create"

https://learn.microsoft.com/en-us/azure/api-management/azure-openai-emit-token-metric-policy

### 2.2 Testowanie dostępności OpenAI API

1. Po utworzeniu API, wybierz je z listy
2. Wybierz operacje "Creates a completion for the chat message"
3. Przejdź do zakładki "Test"
4. Dla deploymentu wybierz "gpt-4o-mini" (lub inny dostępny)
5. Dla api-version ustaw "2024-05-01-preview"
6. W body umieść poniższy JSON:

```json
{
  "messages": [
    {
      "role": "system",
      "content": "Say Hello World"
    }
  ]
}
```

7. Kliknij "Send" i sprawdź odpowiedź
8. Tym razem nie wyłączylismy w zakładce "Settings" opcji "Subscription required", a jednak udało się nam wysłać zapytanie. Dzieje się to dlatego że portal automatycznie podkłada klucz, możesz to sprawdzić poprzez zakładkę "Trace".
### 2.3 Zmiana ustawień subskrypcji dla API polisy-ai

1. Przejdź do "APIs" i wybierz "polisy-ai"
2. Przejdź do zakładki "Settings"
3. W sekcji "Subscription" zaznacz opcję "Subscription required"
4. Upewnij się że w "Header name" wartość to "Ocp-Apim-Subscription-Key" a w "Query parameter name" widnieje wartość "subscription-key"
5. Kliknij "Save"

### 2.4 Dodawanie uwierzytelniania Managed Identity do OpenAI

https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-use-managed-service-identity

Sprawdź czy został włączony dla "API Management" "system managed identity" i czy zostało nadane uprawnienie dla tej tożsamości do "Azure Open AI". "Managed Identity" powinno zostać utworzone podczas tworzenia API Managemnt, rola powinna zostać nadana podczas dodawania API "polisy-ai".

1. Przejdź do swojego API Management
2. W menu bocznym wybierz "Managed identities"
3. Włącz opcję "System assigned" i kliknij "Save"
4. Przejdź do zasobu Azure OpenAI
5. Wybierz "Access control (IAM)"
6. Kliknij "+ Add" i wybierz "Add role assignment"
7. Wybierz rolę "Cognitive Services OpenAI User"
8. W zakładce "Members" wybierz "Managed identity" i wskaż swój APIM
9. Kliknij "Review + assign"

---

## 3. KLUCZE

### 3.1 Konfiguracja kluczy API w Azure API Management

https://learn.microsoft.com/en-us/azure/api-management/api-management-subscriptions

1. W zasobie API Management przejdź do sekcji "Subscriptions"
2. Stwórz nową subskrypcję klikając "+ Add":
    - **Name**: WorkshopSubscription
    - **Scope**: All APIs (lub konkretne API)
3. Po utworzeniu, kliknij na subskrypcję i skopiuj wygenerowany klucz

### 3.2 Włączanie wymogu klucza subskrypcji dla API

1. Przejdź do "APIs" i wybierz "PolisyAPI"
2. Przejdź do zakładki "Settings"
3. W sekcji "Subscription" zaznacz opcję "Subscription required"
4. Upewnij się że w "Header name" wartość to "Ocp-Apim-Subscription-Key" a w "Query parameter name" widnieje wartość "subscription-key"
5. Kliknij "Save"

### 3.3 Testowanie API z kluczem

1. Przejdź do zakładki "Test"
2. Wybierz operację GET /polisy
3. Kliknij "Send" i zweryfikuj, że otrzymujesz prawidłową odpowiedź
4. Sprawdź jak wygląda pełny request (ikonka oka po prawej stronie w sekcji HTTP request). Narzędzie do testowania samo dodaje header "Ocp-Apim-Subscription-Key". Jeśli będziesz korzystał z innych narzędzi pamiętaj o dodaniu header "Ocp-Apim-Subscription-Key" wraz z prawidłowym kluczem.

---

## 4. RATE LIMITS

### 4.1 Konfiguracja Rate Limiting

https://learn.microsoft.com/en-us/azure/api-management/rate-limit-policy

1. Przejdź do "APIs" i wybierz "PolisyAPI"
2. Wybierz zakładkę "Designs", pozostań w "All operations", przejdź do sekcji "Inbound processing", kliknij w </>.
3. W edytorze XML, w sekcji `<inbound>` dodaj za znacznikiem `<base />`:

```xml
<rate-limit calls="5" renewal-period="30" />
```

4. Kliknij "Save"

Ta polityka ogranicza liczbę wywołań do 5 na 30 sekund.

### 4.2 Testowanie ograniczenia liczby wywołań

1. Przejdź do zakładki "Test"
2. Wybierz operację GET /polisy
3. Kliknij "Send" co najmniej 6 razy w ciągu 30 sekund
4. Zauważ, że po 5 wywołaniach otrzymujesz błąd 429 Too Many Requests

### 4.3 Usuwanie Rate Limiting

1. Wróć do zakładki "Policies"
2. W edytorze XML usuń linię

```xml
<rate-limit calls="5" renewal-period="30" />
```

3. Kliknij "Save"

---

## 5. OAUTH 2.0

### 5.1 Rejestracja aplikacji w Microsoft Entra ID

https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app

1. W Azure Portal przejdź do "Microsoft Entra ID"
2. Wybierz "App registrations" i kliknij "+ New registration"
3. Wypełnij formularz:
    - **Name**: PolisyAPI-OAuth
    - **Supported account types**: wybierz "Accounts in this organizational directory only"
4. Kliknij "Register"
5. Zanotuj wartości "Application (client) ID" oraz "Directory (tenant) ID"
6. Wygeneruj secret i zapisz klucz (pamiętaj o zapisaniu klucza po wygenerowaniu – będzie tylko widoczny przez chwilę).

### 5.2 Implementacja polityki uwierzytelniania Azure AD

https://learn.microsoft.com/en-us/azure/api-management/validate-azure-ad-token-policy

1. Przejdź do "Policies" dla "PolisyAPI"
2. W edytorze XML dodaj w sekcji `<inbound>` po `<base />`:

```xml
<policies>
    <inbound>
        <base />
        <validate-azure-ad-token tenant-id="xxxx">
            <client-application-ids>
                <application-id>xxxx</application-id>
            </client-application-ids>
        </validate-azure-ad-token>
    </inbound>
```

3. Zastąp "xxxxxxxxxxx" swoim Tenant ID oraz "xxxxxxxxxx" swoim Client ID
4. Kliknij "Save"

### 5.3 Utwórz prostą Azure LogicApp która pomoże ci przetestować uwierzytelnianie.

https://learn.microsoft.com/en-us/azure/logic-apps/quickstart-create-example-consumption-workflow

1. Na głównej stronie https://portal.azure.com, wybierz opcję "Create a resource".
2. Wyszukaj "Logic App", kliknij "Create"
3. Wybierz "Multi-tenant"
4. Wybierz "Select"
5. Wybierz "Subscription" na której wdrożyłeś API Management
6. Wybierz "Resource Group" na której wdrożyłeś API Management
7. W polu "Logic App name" wpisz "la-test01"
8. W polu "Region" wybierz ten sam region w którym wdrożyłeś API Management.
9. Kliknij "Review + Create" a następnie "Create".
10. Po utworzeniu zasobu kliknij "Go to resource".
11. Kliknij "Edit"
12. Kliknij "Add a trigger", wybierz "Request", następnie "When a HTTP request is received", kliknij "Save".
13. Kliknij znaczek + który znajduje się poniżej kafelka "When a HTTP request is received", wybierz "Add an action"
14. Wyszukaj "Azure API Management" - wybierz "Choose an Azure API Management action", zaznacz swoją instancję Azure API Management.
15. Wybierz "PolisyAPI", kliknij "Add action".
16. W polu "Operation Id" wybierz "GetPolisy".
17. W polu "Advanced parameters" zaznacz zarówno "Authentication" jak i "Subscription key".
18. W polu "Authentication" wybierz Active Directory OAuth a następnie wypełnij wszystkie wymagane pola takie jak "Tenant", "Audience", "Client ID" oraz "Secret", w polu "Audience" wpisz "https://management.azure.com/".
19. W polu "Subscription key" wpisz klucz który wygenerowałeś w punkcie "3.1", kliknij "Save".
20. Kliknij "Run" następnie "Run".
21. Przejdź na "Overview" i sprawdź w zakładce "Run History" wynik wysłania zapytania do "API" wystawionego przez "Azure API Management".
22. Możesz poeksperymentować i pozmieniać wartości np. zmienić klucz na błędny aby sprawdzić że uwierzytelnianie działa. Błędy możesz sprawdzić w "History".

---

## 6. OPEN AI TOKEN LIMIT

### 6.1 Skonfiguruj "Azure Logic App" aby umożliwiało wykorzystanie "Managed Identity" do łączenia się z innymi usługami takimi jak np. "Azure API Management".

1. Przejdź do "Azure Logic App" o nazwie "la-test01".
2. Przejdź do zakładki "Identity", kliknij "System assigned", wybierz "ON" a następnie "Save".
3. W EntraID znajdź "Application ID", który dotyczy "Managed Identity" utworzonego dla "Azure Logic App". Przejdź do "Entra ID" następnie "Enterprise applications", w "Application type" wybierz "Managed Identity", wyszukaj nazwę "la-test01". Zanotuj "Application ID".

### 6.1 Dodawanie polityki limitu tokenów OpenAI

https://learn.microsoft.com/en-us/azure/api-management/azure-openai-token-limit-policy

1. Znajdż "Azure API Managment" w portalu azure, następnie przejdź do "APIs" i wybierz API dla OpenAI o nazwie "polisy-ai"
2. Przejdź do sekcji "Inbound processing" a następnie "Policies", kliknij w oznaczenie </>
3. W edytorze XML dodaj w sekcji `<inbound>` przed `<base />`:

```xml
<azure-openai-token-limit counter-key="@(context.Subscription.Id)" tokens-per-minute="100" estimate-prompt-tokens="true" />
```

4. Dodaj również politykę uwierzytelniania Managed Identity do Azure OpenAI:

https://learn.microsoft.com/en-us/azure/api-management/authentication-managed-identity-policy

```xml
<authentication-managed-identity resource="https://cognitiveservices.azure.com" output-token-variable-name="managed-id-access-token" ignore-error="false" />
<set-header name="Authorization" exists-action="override">
  <value>@("Bearer " + (string)context.Variables["managed-id-access-token"])</value>
</set-header>
```

5. Sprawdź czy jest ustawiony backend OpenAI:

Backend-id musi mieć tą samą nazwę co "Backend name" w zakłade "Backends".

```xml
<set-backend-service id="apim-generated-policy" backend-id="polisy-ai-openai-endpoint" />
```

6. Kliknij "Save"

https://learn.microsoft.com/en-us/azure/api-management/validate-azure-ad-token-policy

8. Zmień politykę "validate-azure-ad-token tenant-id" w celu uwierzytelniania komunikacji tylko z określonego Managed Identity w tym przypadku podłączonego pod Azure Logic App, podaj "application-id" z punktu 6.1.3.

```xml
    <validate-azure-ad-token tenant-id="xxxxxxxxxxx">
      <client-application-ids>
        <application-id>xxxxxxxxxx</application-id>
      </client-application-ids>
    </validate-azure-ad-token>
```

Pełna polityka powinna wyglądać następująco:

```xml
<policies>
  <inbound>
    <base />
    <validate-azure-ad-token tenant-id="xxxxxxxxxxx">
      <client-application-ids>
        <application-id>xxxxxxxxxx</application-id>
      </client-application-ids>
    </validate-azure-ad-token>
    <azure-openai-token-limit counter-key="@(context.Subscription.Id)" tokens-per-minute="10000" estimate-prompt-tokens="true" />
    <set-backend-service id="apim-generated-policy" backend-id="polisy-ai-openai-endpoint" />
    <authentication-managed-identity resource="https://cognitiveservices.azure.com" output-token-variable-name="managed-id-access-token" ignore-error="false" />
    <set-header name="Authorization" exists-action="override">
      <value>@("Bearer " + (string)context.Variables["managed-id-access-token"])</value>
    </set-header>
  </inbound>
  <backend>
    <base />
  </backend>
  <outbound>
    <base />
  </outbound>
  <on-error>
    <base />
  </on-error>
</policies>
````

### 6.2 Dodanie do już istniejącego Azure Logic App kolejnego konektora który umożliwi komunikację z Azure Open AI.

1. Przejdź do Azure Logic App następnie kliknij "Edit".
2. Kliknij na pierwszy element "When a HTTP request is received" w polu "Request Body JSON Schema" wklej poniższy kod

```
{
    "type": "object",
    "properties": {
        "prompt": {
            "type": "string"
        }
    }
}
```

3. Dodaj na sam koniec przepływu (poprzez znaczek +), akcję o nazwie "API Management", wypełnij formularz, wybierz swoje API Management a następnie wybierz polisy-ai API, kliknij "Add action".
4. W polu "Operation Id" wybierz "Creates a completion for the chat message".
5. W polu "Deployment-ID" wpisz "gpt-4o-mini" lub inny model który jest dostępny w Azure Open AI.
6. W polu "api-version" wpisz "2024-05-01-preview".
7. W polu "Body" wpisz

```
{
  "messages": [
    {
      "role": "system",
      "content": "@{outputs('polisy-api')}"
    },
    {
      "role": "user", 
      "content": "@{triggerBody()?['prompt']}"
    }
  ]
}
```

8. W polu "Advanced parameters" zaznacz "Authentication" oraz "Subscription key". W części "Authentication Types" wybierz "Managed identity" w części "Managed identity" wybierz "System-assigned managed identity". W polu "Subscription key" wpisz klucz który wygenerowałeś w punkcie "3.1".
9. Zmień ustawienia pierwszej akcji o nazwie "polisy-api", aby wykorzystywała "System-assigned managed identity".
10. Sprawdź działanie Azure Logic App, wybierz przycisk "Run" a następnie "Run with payload". W sekcji "Body" wprowadź poniższy kod

```
{
    "prompt": "Proszę podać id oraz ceny dotyczące polis ubezpieczeniowych. Napisz którą polise lepiej wybrać?"
}
```

https://learn.microsoft.com/en-us/azure/logic-apps/monitor-logic-apps-overview

10. Poczekaj kilka sekund na odpowiedź z Azure Open AI i kliknij na "View monitoring view", sprawdź jak wyglądał przepływ zdarzeń w Azure Logic App. Przejdź do klocka o nazwie "polisy-ai" i w sekcji "Outputs" znajdź "Body", sprawdź odpowiedź od modelu.

---

## 7. TRANSFORMACJA/ANONIMIZACJA

### 7.1 Stosowanie polityk transformacji

https://learn.microsoft.com/en-us/azure/api-management/json-to-xml-policy

1. Przejdź do "APIs" i wybierz "polisy-ai"
2. Przejdź do "Policies"
3. W edytorze XML, w sekcji `<outbound>` po znaczniku `<base />` dodaj politykę konwersji JSON do XML:

```
        <json-to-xml apply="always" consider-accept-header="false" parse-date="false" />
```

### 7.2 Dodawanie polityki anonimizacji danych

https://learn.microsoft.com/en-us/azure/api-management/find-and-replace-policy

1. Pozostając w edytorze polityk, dodaj w sekcji `<outbound>` po polityce transformacji:

```
        <find-and-replace from="123456" to="xxxxxx" />
```

2. Kliknij "Save"

### 7.3 Testowanie transformacji i anonimizacji v1

1. Uruchom Azure Logic App tak jak w części 6.2.10 i sprawdź że obecnie "Body" na "Outputs" jest w postaci xml, oraz że id polisy zostało zastąpione z 123456 na xxxxxx.

---

### 7.4 Zmiana find-and-replace na RegularExpressions

1. W edytorze polityk zmień 

```
        <find-and-replace from="123" to="xxx" />
```

na

https://learn.microsoft.com/en-us/azure/api-management/api-management-policy-expressions

```
        <set-body>@{
        string body = context.Response.Body.As<string>(preserveContent: true);
        body = System.Text.RegularExpressions.Regex.Replace(body,  @"\b\d{6}\b", "xxxxxx");
        return body;}
        </set-body>
```
2. Kliknij "Save"
3. Zaakceptuj komunikat "Warning".

### 7.5 Testowanie transformacji i anonimizacji v2

1. Uruchom Azure Logic App tak jak w części 6.2.10 i sprawdź że obecnie "Body" na "Outputs" jest w postaci xml, oraz że wszystkie id polisy zostały zastąpione z 123456 oraz 123457 na xxxxxx.

## 8. MONITOROWANIE I DIAGNOSTYKA W APIM

### 8.1 Konfiguracja Application Insights

https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-app-insights?tabs=rest

1. Jeśli nie masz jeszcze zasobu Application Insights, utwórz go:
    - Wyszukaj "Application Insights" w Azure Portal
    - Kliknij "+ Create"
    - Wypełnij formularz i utwórz zasób
2. Przejdź do zasobu API Management
3. W menu bocznym wyszukaj "Application Insights" dodaj wcześniej utworzony zasób Application Insights.
4. Następnie w menu bocznym wybierz "APIs" następnie "All APIs"
5. Kliknij "Settings" i kliknij "Enable" dla "Application Insight"
6. W polu "Destination" wybierz wcześniej utworzony "Application Insight"
7. W polu "Verbosity" zaznacz "Verbose"
8. Kliknij "Save"

### 8.2 Konfiguracja logowania i śledzenia

https://learn.microsoft.com/en-us/azure/api-management/trace-policy

1. Przejdź do "APIs" i wybierz "polisy-ai"
2. Wybierz zakładkę "Policies"
3. W edytorze XML dodaj w sekcji `<inbound>` przed `<base />`:

```xml
        <!--Use consumer correlation id or generate new one-->
        <set-variable name="correlation-id" value="@(context.Request.Headers.GetValueOrDefault("x-ms-client-tracking-id", Guid.NewGuid().ToString()))" />
        <!--Set header for end-to-end correlation-->
        <set-header name="x-correlation-id" exists-action="override">
            <value>@((string)context.Variables["correlation-id"])</value>
        </set-header>
        <trace source="API Management Trace">
            <message>@{
    return "Rozpoczęcie przetwarzania żądania " + context.Request.Method + " " + context.Request.Url.Path;
  }</message>
            <metadata name="User-Agent" value="@(context.Request.Headers.GetValueOrDefault("User-Agent", ""))" />
            <metadata name="Subscription-Id" value="@(context.Subscription?.Id ?? "anonymous")" />
            <metadata name="correlation-id" value="@((string)context.Variables["correlation-id"])" />
        </trace>
```

4. W sekcji `<outbound>` po `<base />` dodaj:

```xml
        <trace source="API Management Trace">
            <message>@{
    return "Zakończenie przetwarzania, status: " + context.Response.StatusCode;
  }</message>
            <metadata name="User-Agent" value="@(context.Request.Headers.GetValueOrDefault("User-Agent", ""))" />
            <metadata name="Subscription-Id" value="@(context.Subscription?.Id ?? "anonymous")" />
            <metadata name="correlation-id" value="@((string)context.Variables["correlation-id"])" />
        </trace>
```

5. Kliknij "Save"

Pełna polityka powinna wyglądać następująco:

```xml
<policies>
    <inbound>
        <base />
        <!--Use consumer correlation id or generate new one-->
        <set-variable name="correlation-id" value="@(context.Request.Headers.GetValueOrDefault("x-ms-client-tracking-id", Guid.NewGuid().ToString()))" />
        <!--Set header for end-to-end correlation-->
        <set-header name="x-correlation-id" exists-action="override">
            <value>@((string)context.Variables["correlation-id"])</value>
        </set-header>
        <trace source="API Management Trace">
            <message>@{
    return "Rozpoczęcie przetwarzania żądania " + context.Request.Method + " " + context.Request.Url.Path;
  }</message>
            <metadata name="User-Agent" value="@(context.Request.Headers.GetValueOrDefault("User-Agent", ""))" />
            <metadata name="Subscription-Id" value="@(context.Subscription?.Id ?? "anonymous")" />
            <metadata name="correlation-id" value="@((string)context.Variables["correlation-id"])" />
        </trace>
        <validate-azure-ad-token tenant-id="xxxxxxxxxxx">
            <client-application-ids>
                <application-id>xxxxxxxxxxx</application-id>
            </client-application-ids>
        </validate-azure-ad-token>
        <azure-openai-token-limit counter-key="@(context.Subscription.Id)" tokens-per-minute="10000" estimate-prompt-tokens="true" />
        <set-backend-service id="apim-generated-policy" backend-id="polisy-ai-openai-endpoint" />
        <authentication-managed-identity resource="https://cognitiveservices.azure.com" output-token-variable-name="managed-id-access-token" ignore-error="false" />
        <set-header name="Authorization" exists-action="override">
            <value>@("Bearer " + (string)context.Variables["managed-id-access-token"])</value>
        </set-header>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
        <trace source="API Management Trace">
            <message>@{
    return "Zakończenie przetwarzania, status: " + context.Response.StatusCode;
  }</message>
            <metadata name="User-Agent" value="@(context.Request.Headers.GetValueOrDefault("User-Agent", ""))" />
            <metadata name="Subscription-Id" value="@(context.Subscription?.Id ?? "anonymous")" />
            <metadata name="correlation-id" value="@((string)context.Variables["correlation-id"])" />
        </trace>
        <json-to-xml apply="always" consider-accept-header="false" parse-date="false" />
        <set-body>@{
        string body = context.Response.Body.As<string>(preserveContent: true);
        body = System.Text.RegularExpressions.Regex.Replace(body,  @"\b\d{6}\b", "xxxxxx");
        return body;}</set-body>
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

### 8.3 Analiza metryk i logów

https://learn.microsoft.com/en-us/azure/azure-monitor/app/transaction-search-and-diagnostics?tabs=transaction-search

1. Wykonaj kilka zapytań do API
2. Przejdź do zasobu Application Insights
3. W menu bocznym wybierz "Investigate" a następnie "Transaction search"
4. Sprawdź jak wyglądają wyniki.

---

## 9. Zapoznaj się z innymi politykami

Na stronie https://learn.microsoft.com/en-us/azure/api-management/api-management-policies możesz zapoznać się z pełną listą polityk dostępnych w Azure API Management. Warto abyś sprawdził polityki takie jak Caching czy np. Rewrite URL. Dla Azure Open AI warto również zapoznać się z informacjami o semantic caching, znajdziesz więcej informacji na tej stronie: https://learn.microsoft.com/en-us/azure/api-management/azure-openai-enable-semantic-caching

---

## Podsumowanie

Gratulacje! Stworzyłeś kompletny interfejs API za pomocą Azure API Management, który:

- Dostarcza informacje o polisach ubezpieczeniowych
- Integruje się z Azure OpenAI Service
- Jest zabezpieczony przez klucze API i OAuth 2.0
- Kontroluje ruch za pomocą limitów wywołań
- Limit tokenów dla zapytań AI
- Transformuje i anonimizuje dane

- Jest monitorowany w Application Insights
