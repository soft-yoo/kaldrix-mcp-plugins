# Kaldrix plugins for AI assistants

[![M8ven Score](https://m8ven.ai/badge/mcp/soft-yoo-kaldrix-mcp-plugins-xzklay)](https://m8ven.ai/mcp/soft-yoo-kaldrix-mcp-plugins-xzklay?s=readme)

[English](#kaldrix-plugins-for-ai-assistants) · [Polski](#pluginy-kaldrix-dla-asystentów-ai)

Manage construction projects, prepare cost estimates and plan work in Kaldrix with your
AI assistant. This repository provides installation packages for Claude Code, Gemini CLI,
ChatGPT and Codex. All packages connect to the same Kaldrix service at `https://api.kaldrix.pl/mcp`.

The badge above opens M8ven's current report for this package repository.

## Before you start

- **Create a Kaldrix account.** Sign in to Kaldrix when your assistant first connects.
- **Choose your assistant below.** Each client has its own package and installation steps.
- **Check your subscription.** The plugin is free. Reading your data does not require an active
  subscription; creating or changing data requires an active Kaldrix subscription.

## Enable AI connections in Kaldrix first

1. Sign in to [Kaldrix](https://app.kaldrix.pl/ai) and check that you have selected the right company.
   In **Subscription** → **Features**, check that the **AI** module is on.
   The company owner can enable it if the plan includes AI.
2. Open **AI** → **Connect an AI tool** and turn on
   **Allow connecting AI tools to Kaldrix in this company**.
   Only someone with **Company details and settings: Full access** can change this setting.
   If you do not have that access, ask a person in your company who does.
3. Keep connections on, then install the package for your assistant below.
   When asked for a server address, use **Copy address** on the same Kaldrix page.

The AI module and connection setting apply to the whole company. When either is off,
new connections and requests from already connected assistants are blocked. Each person
still signs in with their own Kaldrix account and uses only the access they have been granted.

If you sign in with a password, first enable the phone code in **My Profile** → **Security** →
**Turn on the phone code**. It is required for AI connections. Google sign-in does not need this extra step.

## Download and install

Choose the assistant you use, then follow its installation guide. The download links
always point to the newest [release](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases).

| Assistant | Installation guide | Download ZIP |
| --- | --- | --- |
| Claude Code | [Install in Claude Code](#claude) | [kaldrix-claude.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-claude.zip) |
| Gemini CLI | [Install in Gemini CLI](#gemini-cli) | [kaldrix-gemini.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-gemini.zip) |
| ChatGPT and Codex | [Install in ChatGPT or Codex](#chatgpt-and-codex) | [kaldrix-chatgpt.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-chatgpt.zip) |

Use the package for your assistant. `SHA256SUMS` in each release lists the checksums:
`sha256sum -c SHA256SUMS --ignore-missing`.
The `darwin.`, `linux.` and `win32.` copies of the Gemini package are the same file;
Gemini CLI picks one of them when it installs from this repository.

## What you can do

- **Quote a house in a few steps.** Describe the building (size, storeys, walls, roof,
  foundations). Kaldrix creates the model and a draft cost estimate from your price list.
- **Model in 2D and 3D.** Walls, rooms, doors and windows, roofs and roof framing,
  structures and installations. Kaldrix checks the model after every change.
- **Review estimates before you share them.** Totals, VAT, markups and warnings;
  service recipes and price lists.
- **Run the project.** Offers and client remarks, contracts and annexes, the schedule,
  the site log, acceptance and handover.
- **Keep the office in order.** Customers, tasks, documents and received invoices.

Kaldrix validates and saves every change. Deleting data, sending documents to a client
and replacing prices need your explicit confirmation. The assistant can do only what
your permissions in Kaldrix and the access you grant when connecting allow.

## Try asking

- "Find my project Kowalski house and tell me what should happen next."
- "Prepare a quote for a 10 by 8 metre single-storey masonry house with strip footings and a gable roof."
- "Check the cost estimate for Kowalski house and explain the warnings before I send it."
- "Prepare a draft contract from the accepted offer. Do not send it."
- "Record this supplier invoice and keep the attached PDF."

## Claude

### Claude Code (terminal)

1. Add this repository and install the plugin:

   ```bash
   claude plugin marketplace add soft-yoo/kaldrix-mcp-plugins
   claude plugin install kaldrixprod@kaldrix
   ```

   Inside a Claude Code session (version 2.1.275 or later) you can do both at once:
   `/plugin install kaldrixprod --marketplace soft-yoo/kaldrix-mcp-plugins`.
2. Start a new session (or run `/reload-plugins`). Run `/mcp`, choose `kaldrix`
   and sign in to Kaldrix in the browser window. Pick your company if you have more than one.
3. Ask, for example, *"Show my Kaldrix projects"* or *"Review the cost estimate of project X"*.

Update: `claude plugin update kaldrixprod@kaldrix`. Remove: `claude plugin uninstall kaldrixprod@kaldrix`.

The marketplace is named `kaldrix`, like the one in
[soft-yoo/kaldrix-claude-plugin](https://github.com/soft-yoo/kaldrix-claude-plugin).
Add only one of them; both install the same plugin.

To try the plugin for one session without installing it:

```bash
claude --plugin-url https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-claude.zip
```

### claude.ai, Claude Desktop and Cowork

These apps install plugins from Anthropic's directory. Kaldrix is waiting for
Anthropic's review; after approval you will add it from
[claude.ai/directory](https://claude.ai/directory). Supported Claude Code versions can sync it
when you sign in with the same Claude account.
The ZIP from this repository is for Claude Code.

## Gemini CLI

1. Install the extension from this repository and confirm the prompt:

   ```bash
   gemini extensions install https://github.com/soft-yoo/kaldrix-mcp-plugins
   ```

2. Start `gemini`, run `/mcp auth kaldrix` and sign in to Kaldrix in the browser window.
   `/mcp` shows the connection and its tools.
3. Ask, for example, *"Show my Kaldrix projects"*.

Update: `gemini extensions update kaldrixprod`. Remove: `gemini extensions uninstall kaldrixprod`.
The repository root also includes `gemini-extension.json` and `skills/` for the
[Gemini CLI extensions gallery](https://geminicli.com/extensions/).
On systems other than macOS, Linux and Windows, download `kaldrix-gemini.zip`,
unpack it and run `gemini extensions install <unpacked folder>`.

## ChatGPT and Codex

### ChatGPT (web)

Your ChatGPT account or workspace must allow uploading plugins with custom MCP servers.
In a managed workspace, ask your administrator if the required options are unavailable.
See [OpenAI's plugin permissions](https://learn.chatgpt.com/docs/enterprise/apps-and-connectors#choose-what-members-can-do-with-plugins).

1. Download [kaldrix-chatgpt.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-chatgpt.zip).
2. In ChatGPT on the web open **Settings** and turn on **Developer mode**.
3. Open **Plugins**, select **Add** → **Upload plugin archive** and choose `kaldrix-chatgpt.zip`.
4. When ChatGPT asks for the MCP server, enter `https://api.kaldrix.pl/mcp`
   and sign in to Kaldrix in the window that opens. Pick your company if you have more than one.
5. Install Kaldrix from your plugins and ask, for example, *"Show my Kaldrix projects"*.

The uploaded plugin is visible only to you. A workspace admin or owner can publish it
to the workspace. To update, upload the newest `kaldrix-chatgpt.zip` again.
After OpenAI approves Kaldrix, you will also find it in the ChatGPT plugin directory.

### Codex

1. Unpack `kaldrix-chatgpt.zip` into `~/.codex/plugins/kaldrixprod`.
2. Add an entry for that folder to your personal marketplace `~/.agents/plugins/marketplace.json`,
   as described in [OpenAI: build plugins](https://developers.openai.com/plugins/build/plugins).
3. Restart Codex, install Kaldrix from the plugin list and sign in to Kaldrix when asked.

## Where your data goes

- **Kaldrix MCP server** (`https://api.kaldrix.pl/mcp`): every read and change of your
  Kaldrix data, for example projects, cost estimates, customers (names, emails, addresses),
  invoices and contracts. Data you save stays in your company account until you delete it
  or delete the account.
- **Kaldrix file storage** (`https://s3.waw.io.cloud.ovh.net`, OVH, Warsaw): large files you
  ask the assistant to upload (documents, textures, 3D models) are sent with HTTP PUT to a one-time
  upload address that the Kaldrix server returns. Small files go through the MCP server.
- Sign-in uses OAuth in your AI client. The packages contain no passwords or tokens
  and send no other data.

This repository is generated from the Kaldrix sources; changes made here are overwritten.

Help: [kaldrix.pl/integracje.html](https://kaldrix.pl/integracje.html) ·
Privacy: [kaldrix.pl/app-privacy.html](https://kaldrix.pl/app-privacy.html) ·
Terms: [kaldrix.pl/app-terms.html](https://kaldrix.pl/app-terms.html)

---

# Pluginy Kaldrix dla asystentów AI

[English](#kaldrix-plugins-for-ai-assistants) · [Polski](#pluginy-kaldrix-dla-asystentów-ai)

Prowadź projekty budowy, przygotowuj kosztorysy i planuj pracę w Kaldrix z pomocą
asystenta AI. W tym repozytorium znajdziesz paczki do instalacji w Claude Code,
Gemini CLI, ChatGPT i Codex. Każda łączy się z Kaldrix pod adresem `https://api.kaldrix.pl/mcp`.

Odznaka na początku strony prowadzi do aktualnego raportu M8ven o tym repozytorium paczek.

## Zanim zaczniesz

- **Załóż konto Kaldrix.** Zaloguj się do niego przy pierwszym połączeniu asystenta.
- **Wybierz swojego asystenta poniżej.** Każdy program ma własną paczkę i instrukcję instalacji.
- **Sprawdź subskrypcję.** Plugin jest bezpłatny. Przeglądanie danych nie wymaga aktywnej
  subskrypcji; tworzenie i zmiana danych wymagają aktywnej subskrypcji Kaldrix.

## Najpierw włącz połączenia AI w Kaldrix

1. Zaloguj się do [Kaldrix](https://app.kaldrix.pl/ai) i sprawdź, czy masz wybraną właściwą firmę.
   W **Subskrypcja** → **Funkcje** sprawdź, czy moduł **AI** jest włączony.
   Właściciel firmy może go włączyć, jeśli plan obejmuje AI.
2. Otwórz **AI** → **Podłącz narzędzie AI** i włącz
   **Pozwól łączyć narzędzia AI z Kaldrix w tej firmie**.
   Przełącznik może zmienić tylko osoba z uprawnieniem **Dane i ustawienia firmy: Pełny dostęp**.
   Jeśli nie masz tego uprawnienia, poproś osobę w firmie, która je ma.
3. Pozostaw połączenia włączone i zainstaluj poniżej paczkę dla swojego asystenta.
   Gdy program zapyta o adres serwera, użyj **Kopiuj adres** na tej samej stronie Kaldrix.

Moduł AI i zgoda na połączenia dotyczą całej firmy. Wyłączenie któregokolwiek z nich
blokuje nowe połączenia oraz żądania od już połączonych asystentów. Każda osoba nadal
loguje się na własne konto Kaldrix i korzysta wyłącznie z nadanego jej dostępu.

Jeśli logujesz się hasłem, najpierw włącz kod z telefonu w **Mój profil** → **Bezpieczeństwo** →
**Włącz kod z telefonu**. Jest wymagany przy łączeniu AI. Przy logowaniu przez Google ten dodatkowy krok nie jest potrzebny.

## Pobieranie i instalacja

Wybierz program, którego używasz, i przejdź do jego instrukcji. Linki do pobrania
prowadzą do najnowszego [wydania](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases).

| Asystent | Instrukcja instalacji | Pobierz ZIP |
| --- | --- | --- |
| Claude Code | [Instalacja w Claude Code](#instalacja-w-claude-code) | [kaldrix-claude.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-claude.zip) |
| Gemini CLI | [Instalacja w Gemini CLI](#instalacja-w-gemini-cli) | [kaldrix-gemini.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-gemini.zip) |
| ChatGPT i Codex | [Instalacja w ChatGPT lub Codex](#instalacja-w-chatgpt-i-codex) | [kaldrix-chatgpt.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-chatgpt.zip) |

Pobierz paczkę przeznaczoną dla swojego asystenta. Sumy kontrolne są w pliku `SHA256SUMS`
każdego wydania. Sprawdzisz je poleceniem `sha256sum -c SHA256SUMS --ignore-missing`.
Kopie paczki Gemini z prefiksami `darwin.`, `linux.` i `win32.` mają identyczną zawartość;
Gemini CLI wybiera odpowiednią nazwę przy instalacji z repozytorium.

## Co możesz zrobić

- wycenić dom w kilku krokach: podajesz wymiary, kondygnacje, ściany, dach i fundamenty,
  a Kaldrix tworzy model i szkic kosztorysu z Twojego cennika;
- rysować i poprawiać model 2D i 3D: ściany, pomieszczenia, okna i drzwi, dach z więźbą,
  konstrukcje i instalacje; Kaldrix sprawdza model po każdej zmianie;
- sprawdzić kosztorys przed wysłaniem: sumy, VAT, narzuty i ostrzeżenia, a także usługi i cenniki;
- prowadzić budowę: oferty i uwagi klienta, umowy i aneksy, harmonogram, dziennik budowy i odbiór;
- uporządkować biuro: klienci, zadania, dokumenty i faktury kosztowe.

Kaldrix sprawdza i zapisuje każdą zmianę. Usunięcie danych, wysłanie dokumentu do klienta
i zastąpienie cen wymagają Twojego wyraźnego potwierdzenia. Asystent działa w granicach
Twoich uprawnień w Kaldrix i dostępu, który nadasz przy połączeniu.

## Przykładowe polecenia

- „Znajdź projekt Dom Kowalskich i powiedz, co trzeba teraz zrobić.”
- „Przygotuj wycenę parterowego domu murowanego 10 na 8 m z ławami i dachem dwuspadowym.”
- „Sprawdź kosztorys Domu Kowalskich i wyjaśnij ostrzeżenia, zanim go wyślę.”
- „Przygotuj projekt umowy z zaakceptowanej oferty. Nie wysyłaj jej.”
- „Zapisz tę fakturę od dostawcy i zachowaj załączony PDF.”

## Instalacja w Claude Code

1. Dodaj to repozytorium i zainstaluj plugin:

   ```bash
   claude plugin marketplace add soft-yoo/kaldrix-mcp-plugins
   claude plugin install kaldrixprod@kaldrix
   ```

   W otwartej sesji Claude Code (od wersji 2.1.275) możesz zrobić to jednym poleceniem:
   `/plugin install kaldrixprod --marketplace soft-yoo/kaldrix-mcp-plugins`.
2. Otwórz nową sesję (albo wpisz `/reload-plugins`). Wpisz `/mcp`, wybierz `kaldrix`
   i zaloguj się do Kaldrix w oknie przeglądarki. Jeśli masz kilka firm, wybierz właściwą.
3. Poproś na przykład: *„Pokaż moje projekty w Kaldrix”* albo *„Sprawdź kosztorys projektu X”*.

Aktualizacja: `claude plugin update kaldrixprod@kaldrix`. Usunięcie: `claude plugin uninstall kaldrixprod@kaldrix`.

Repozytorium [soft-yoo/kaldrix-claude-plugin](https://github.com/soft-yoo/kaldrix-claude-plugin)
instaluje ten sam plugin pod tą samą nazwą `kaldrix`. Dodaj tylko jedno z nich.

Aby wypróbować plugin w jednej sesji bez instalacji:

```bash
claude --plugin-url https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-claude.zip
```

## claude.ai, Claude Desktop i Cowork

Te aplikacje instalują pluginy z katalogu Anthropic. Kaldrix czeka na sprawdzenie
przez Anthropic. Po zatwierdzeniu dodasz go z [claude.ai/directory](https://claude.ai/directory).
Obsługiwane wersje Claude Code mogą go też pobrać, gdy zalogujesz się na to samo konto Claude.
ZIP z tego repozytorium jest dla Claude Code.

## Instalacja w Gemini CLI

1. Zainstaluj rozszerzenie z tego repozytorium i potwierdź pytanie:

   ```bash
   gemini extensions install https://github.com/soft-yoo/kaldrix-mcp-plugins
   ```

2. Uruchom `gemini`, wpisz `/mcp auth kaldrix` i zaloguj się do Kaldrix w oknie przeglądarki.
   `/mcp` pokazuje połączenie i dostępne narzędzia.
3. Poproś na przykład: *„Pokaż moje projekty w Kaldrix”*.

Aktualizacja: `gemini extensions update kaldrixprod`. Usunięcie: `gemini extensions uninstall kaldrixprod`.
W korzeniu repozytorium znajdują się też `gemini-extension.json` i `skills/` przygotowane dla
[galerii rozszerzeń Gemini CLI](https://geminicli.com/extensions/).
Na systemach innych niż macOS, Linux i Windows pobierz `kaldrix-gemini.zip`,
rozpakuj go i wpisz `gemini extensions install <rozpakowany folder>`.

## Instalacja w ChatGPT i Codex

### ChatGPT (w przeglądarce)

Twoje konto lub przestrzeń robocza ChatGPT musi pozwalać na dodawanie pluginów z własnymi serwerami MCP.
Jeśli w przestrzeni firmowej nie widzisz potrzebnych opcji, poproś administratora o sprawdzenie dostępu.
Zobacz [uprawnienia pluginów w dokumentacji OpenAI](https://learn.chatgpt.com/docs/enterprise/apps-and-connectors#choose-what-members-can-do-with-plugins).

1. Pobierz [kaldrix-chatgpt.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-chatgpt.zip).
2. W ChatGPT w przeglądarce otwórz **Settings** i włącz **Developer mode**.
3. Otwórz **Plugins**, wybierz **Add** → **Upload plugin archive** i wskaż `kaldrix-chatgpt.zip`.
4. Gdy ChatGPT zapyta o serwer MCP, wpisz `https://api.kaldrix.pl/mcp`
   i zaloguj się do Kaldrix w otwartym oknie. Jeśli masz kilka firm, wybierz właściwą.
5. Zainstaluj Kaldrix ze swoich pluginów i poproś na przykład: *„Pokaż moje projekty w Kaldrix”*.

Wgrany plugin widzisz tylko Ty. Administrator lub właściciel przestrzeni roboczej może
go opublikować dla całej przestrzeni. Aby zaktualizować, wgraj ponownie najnowszy `kaldrix-chatgpt.zip`.
Po zatwierdzeniu przez OpenAI Kaldrix będzie też w katalogu pluginów ChatGPT.

### Codex

1. Rozpakuj `kaldrix-chatgpt.zip` do folderu `~/.codex/plugins/kaldrixprod`.
2. Dodaj ten folder do osobistej listy pluginów `~/.agents/plugins/marketplace.json`,
   tak jak opisuje [instrukcja OpenAI](https://developers.openai.com/plugins/build/plugins).
3. Uruchom ponownie Codex, zainstaluj Kaldrix z listy pluginów i zaloguj się do Kaldrix.

## Dokąd trafiają dane

- **Serwer Kaldrix** (`https://api.kaldrix.pl/mcp`): wszystkie odczyty i zmiany danych,
  na przykład projekty, kosztorysy, dane klientów (imiona, nazwiska, e-maile, adresy),
  faktury i umowy. Zapisane dane zostają na koncie firmy, dopóki ich nie usuniesz
  lub nie usuniesz konta.
- **Magazyn plików Kaldrix** (`https://s3.waw.io.cloud.ovh.net`, OVH, Warszawa): duże pliki,
  które zlecisz dodać, na przykład dokumenty, tekstury i modele 3D. Asystent wysyła je metodą
  HTTP PUT na jednorazowy adres otrzymany od Kaldrix. Małe pliki przechodzą przez serwer Kaldrix.
- Logowanie odbywa się przez OAuth w Twoim programie z asystentem. Paczki nie zawierają haseł
  ani tokenów i nie wysyłają innych danych.

Repozytorium powstaje automatycznie ze źródeł Kaldrix. Zmiany wprowadzone bezpośrednio tutaj
zostaną nadpisane przy aktualizacji.

Pomoc: [kaldrix.pl/integracje.html](https://kaldrix.pl/integracje.html) ·
Prywatność: [kaldrix.pl/app-privacy.html](https://kaldrix.pl/app-privacy.html) ·
Warunki: [kaldrix.pl/app-terms.html](https://kaldrix.pl/app-terms.html)

## License

[MIT](https://github.com/soft-yoo/kaldrix-mcp-plugins/blob/main/LICENSE). The license covers the files in this repository and its releases.
The Kaldrix service and its MCP server are not part of it.

Licencja MIT obejmuje pliki w tym repozytorium i jego wydaniach. Usługa Kaldrix
i jej serwer MCP nie są objęte tą licencją.
