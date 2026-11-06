# Kaldrix plugins for AI assistants

Work with your Kaldrix construction projects, schedules and cost estimates in Claude,
Gemini CLI and ChatGPT. Every package connects to the Kaldrix MCP server
(`https://api.kaldrix.pl/mcp`) and adds the same guided workflows: project review,
cost estimate review, service estimation, construction workflow and building model authoring.

You need a Kaldrix account. You sign in to Kaldrix when the plugin first connects;
the packages store no passwords or tokens. The plugin is free. Reading your data
is always available; creating or changing data requires an active Kaldrix subscription.

[Polska wersja poniżej](#pluginy-kaldrix-dla-asystentów-ai).

## Download

Each [release](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases) has one package per client.
The links below always point at the newest release.

| Client | Package | How to use it |
| --- | --- | --- |
| Claude Code | [kaldrix-claude.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-claude.zip) | [Claude](#claude) |
| Gemini CLI | [kaldrix-gemini.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-gemini.zip) | [Gemini CLI](#gemini-cli) |
| ChatGPT and Codex | [kaldrix-chatgpt.zip](https://github.com/soft-yoo/kaldrix-mcp-plugins/releases/latest/download/kaldrix-chatgpt.zip) | [ChatGPT and Codex](#chatgpt-and-codex) |

Packages for different clients are not interchangeable. `SHA256SUMS` in each release
lists the checksums: `sha256sum -c SHA256SUMS --ignore-missing`.
The `darwin.`, `linux.` and `win32.` copies of the Gemini package are the same file;
Gemini CLI picks one of them when it installs from this repository.

## Claude

### Claude Code (terminal)

1. Add this repository and install the plugin:

   ```bash
   claude plugin marketplace add soft-yoo/kaldrix-mcp-plugins
   claude plugin install kaldrixprod@kaldrix
   ```

   Inside a Claude Code session you can do both at once:
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
[claude.ai/directory](https://claude.ai/directory) and it will also load in your Claude Code sessions.
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
On systems other than macOS, Linux and Windows, download `kaldrix-gemini.zip`,
unpack it and run `gemini extensions install <unpacked folder>`.

## ChatGPT and Codex

`kaldrix-chatgpt.zip` is the OpenAI plugin package. It is not a ChatGPT upload file
for everyone: ChatGPT users will add Kaldrix from the ChatGPT plugin directory
after OpenAI approves it.

Until then, on your own computer with the ChatGPT desktop app or Codex:

1. Unpack `kaldrix-chatgpt.zip` into `~/.codex/plugins/kaldrixprod`.
2. Add an entry for that folder to your personal marketplace `~/.agents/plugins/marketplace.json`,
   as described in [OpenAI: build plugins](https://developers.openai.com/plugins/build/plugins).
3. Restart the ChatGPT desktop app, install Kaldrix from the plugin list and sign in to Kaldrix when asked.

A ChatGPT workspace admin can then share the local plugin with the workspace
(ChatGPT Plugins → Personal → ⋯ → Publish). This does not list it in the public directory.

## Where your data goes

- **Kaldrix MCP server** (`https://api.kaldrix.pl/mcp`): every read and change of your
  Kaldrix data, for example projects, cost estimates, customers (names, emails, addresses),
  invoices and contracts. Data you save stays in your company account until you delete it
  or delete the account.
- **Kaldrix file storage** (`https://s3.waw.io.cloud.ovh.net`, OVH, Warsaw): large files you
  ask the assistant to upload (documents, textures, 3D models) are sent with HTTP PUT to a one-time
  upload address that the Kaldrix server returns. Small files go through the MCP server.
- Sign-in uses OAuth in your AI client. The packages send no other data.

This repository is generated from the Kaldrix sources; changes made here are overwritten.

Help: [kaldrix.pl/integracje.html](https://kaldrix.pl/integracje.html) ·
Privacy: [kaldrix.pl/app-privacy.html](https://kaldrix.pl/app-privacy.html) ·
Terms: [kaldrix.pl/app-terms.html](https://kaldrix.pl/app-terms.html)

---

# Pluginy Kaldrix dla asystentów AI

Pracuj z projektami budowy, harmonogramami i kosztorysami Kaldrix w Claude, Gemini CLI
i ChatGPT. Potrzebujesz konta Kaldrix. Przy pierwszym połączeniu logujesz się do Kaldrix;
paczki nie zapisują haseł ani tokenów. Plugin jest bezpłatny. Przeglądanie danych
działa zawsze, a tworzenie i zmiana danych wymaga aktywnej subskrypcji Kaldrix.

Najnowsze paczki pobierzesz z tabeli [Download](#download). Każdy program ma własną paczkę;
nie można ich zamieniać.

## Claude Code (terminal)

1. Dodaj to repozytorium i zainstaluj plugin:

   ```bash
   claude plugin marketplace add soft-yoo/kaldrix-mcp-plugins
   claude plugin install kaldrixprod@kaldrix
   ```

   W otwartej sesji Claude Code możesz zrobić to jednym poleceniem:
   `/plugin install kaldrixprod --marketplace soft-yoo/kaldrix-mcp-plugins`.
2. Otwórz nową sesję (albo wpisz `/reload-plugins`). Wpisz `/mcp`, wybierz `kaldrix`
   i zaloguj się do Kaldrix w oknie przeglądarki. Jeśli masz kilka firm, wybierz właściwą.
3. Poproś na przykład: *„Pokaż moje projekty w Kaldrix”* albo *„Sprawdź kosztorys projektu X”*.

Aktualizacja: `claude plugin update kaldrixprod@kaldrix`. Usunięcie: `claude plugin uninstall kaldrixprod@kaldrix`.

Repozytorium [soft-yoo/kaldrix-claude-plugin](https://github.com/soft-yoo/kaldrix-claude-plugin)
instaluje ten sam plugin pod tą samą nazwą `kaldrix`. Dodaj tylko jedno z nich.

## claude.ai, Claude Desktop i Cowork

Te aplikacje instalują pluginy z katalogu Anthropic. Kaldrix czeka na sprawdzenie
przez Anthropic. Po zatwierdzeniu dodasz go z [claude.ai/directory](https://claude.ai/directory)
i zadziała też w Claude Code. ZIP z tego repozytorium jest dla Claude Code.

## Gemini CLI

1. Zainstaluj rozszerzenie z tego repozytorium i potwierdź pytanie:

   ```bash
   gemini extensions install https://github.com/soft-yoo/kaldrix-mcp-plugins
   ```

2. Uruchom `gemini`, wpisz `/mcp auth kaldrix` i zaloguj się do Kaldrix w oknie przeglądarki.
   `/mcp` pokazuje połączenie i dostępne narzędzia.
3. Poproś na przykład: *„Pokaż moje projekty w Kaldrix”*.

Aktualizacja: `gemini extensions update kaldrixprod`. Usunięcie: `gemini extensions uninstall kaldrixprod`.
Na systemach innych niż macOS, Linux i Windows pobierz `kaldrix-gemini.zip`,
rozpakuj go i wpisz `gemini extensions install <rozpakowany folder>`.

## ChatGPT i Codex

`kaldrix-chatgpt.zip` to paczka pluginu OpenAI. Nie wgrywa się jej w zwykłym ChatGPT:
użytkownicy ChatGPT dodadzą Kaldrix z katalogu pluginów ChatGPT, gdy OpenAI go zatwierdzi.

Do tego czasu na własnym komputerze z aplikacją ChatGPT lub Codex:

1. Rozpakuj `kaldrix-chatgpt.zip` do folderu `~/.codex/plugins/kaldrixprod`.
2. Dodaj ten folder do osobistej listy pluginów `~/.agents/plugins/marketplace.json`,
   tak jak opisuje [instrukcja OpenAI](https://developers.openai.com/plugins/build/plugins).
3. Uruchom ponownie aplikację ChatGPT, zainstaluj Kaldrix z listy pluginów i zaloguj się do Kaldrix.

Administrator przestrzeni roboczej ChatGPT może potem udostępnić plugin całej przestrzeni
(ChatGPT Plugins → Personal → ⋯ → Publish). Nie trafia on wtedy do publicznego katalogu.

**Dokąd trafiają dane:** wszystkie odczyty i zmiany danych idą do serwera MCP Kaldrix
(`https://api.kaldrix.pl/mcp`). Duże pliki (dokumenty, tekstury, modele 3D) asystent wysyła
metodą PUT na jednorazowy adres magazynu plików Kaldrix w OVH (`https://s3.waw.io.cloud.ovh.net`),
który zwraca serwer Kaldrix. Zapisane dane zostają na koncie firmy, dopóki ich nie usuniesz.

## License

[MIT](https://github.com/soft-yoo/kaldrix-mcp-plugins/blob/main/LICENSE). The license covers the files in this repository and its releases.
The Kaldrix service and its MCP server are not part of it.
