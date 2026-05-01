# OCIP: Core Architecture Principles

[cite_start]W Open Composable Integration Platform (OCIP) zasady architektoniczne są ważniejsze niż wybór konkretnych technologii [cite: 168-169]. To one definiują platformę jako spójny produkt klasy Enterprise, a nie tylko zbiór narzędzi Open Source.

Poniższe fundamenty są niepodlegającym negocjacjom trzonem (Tier A) naszej architektury.

## 1. API-First & Event-Driven by Default
Nie łączymy systemów punkt-punkt. [cite_start]Każda integracja musi być od początku projektowana jako niezależny produkt API lub kontrakt zdarzeniowy (Event Contract) [cite: 171-172, 174-175].
* [cite_start]Preferujemy komunikację asynchroniczną (async integration) nad synchroniczną orkiestracją wszędzie tam, gdzie to możliwe [cite: 176-177].
* [cite_start]Wzorzec REST nie może być traktowany jako domyślne rozwiązanie dla każdego problemu integracyjnego[cite: 178].

## 2. Stateless Integration Runtime
[cite_start]Środowisko uruchomieniowe (np. Apache Camel) musi być całkowicie bezstanowe (stateless) [cite: 179-180]. 
* [cite_start]Kategorycznie zabrania się przechowywania stanu biznesowego wewnątrz instancji silnika integracyjnego[cite: 186].
* [cite_start]Cały stan (state) musi być delegowany na zewnątrz: do brokerów wiadomości, baz danych, systemów cache lub dedykowanych silników workflow (np. Temporal) [cite: 181-185].

## 3. Separacja Control Plane i Data Plane
[cite_start]To najważniejszy wzorzec architektoniczny w OCIP [cite: 389-391]. 
* [cite_start]**Data Plane:** Odpowiada wyłącznie za uruchamianie integracji i przepływ danych (obejmuje m.in. Apache Camel, Kafka, API Gateway) [cite: 392-395].
* [cite_start]**Control Plane:** Warstwa nadrzędna, która zarządza platformą (obejmuje Governance, Policies, Deployment, Observability, Templates) [cite: 396-399]. [cite_start]To ta warstwa stanowi faktyczny "produkt" [cite: 400-401].

## 4. Replaceable Infrastructure (Stable Contracts)
[cite_start]Kompozytowa natura OCIP polega na integracji kontraktów, a nie konkretnych narzędzi [cite: 491-494].
* [cite_start]Nie przywiązujemy się do pojedynczej technologii (np. twardy wymóg użycia Apache Kafka) [cite: 215-216, 404-405].
* [cite_start]Przywiązujemy się do zdefiniowanych kontraktów (np. *Event Backbone Contract*), które wymagają określonych funkcji (obsługa DLQ, replay, pub/sub) [cite: 217, 499-506]. [cite_start]Jeśli inny broker klienta spełnia ten kontrakt, platforma może go obsłużyć [cite: 509-510].

## 5. Infrastructure as Code (IaC) & Golden Path
Platforma nie oferuje pełnej, chaotycznej dowolności wyboru ("opcji"). [cite_start]Dostarcza sprawdzoną "Złotą Ścieżkę" (Golden Path) — właściwy sposób działania wymuszony przez standardy platformy [cite: 196-199].
* [cite_start]Konfiguracja manualna jest niedopuszczalna (Zero ręcznej konfiguracji) [cite: 187-188].
* [cite_start]Cały cykl życia platformy (provisioning, deployment, polityki bezpieczeństwa, observability) musi być zdefiniowany deklaratywnie jako kod [cite: 189-195].

## 6. Security & Observability by Default
Mechanizmy zapewniające widoczność i bezpieczeństwo nie są modułami opcjonalnymi.
* [cite_start]**Security:** Każdy przepływ musi mieć domyślnie wbudowane uwierzytelnianie, audytowalność i zarządzanie sekretami [cite: 200-207].
* [cite_start]**Observability:** Każda integracja automatycznie eksponuje metryki, logi i tracing, umożliwiając widoczność SLA [cite: 208-214].
