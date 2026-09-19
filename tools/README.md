# Strumenti per il corpus (`tools`)

Questa cartella contiene l'eseguibile precompilato `corpus-build` utilizzato dal workflow di GitHub Actions `corpus-refresh` per mantenere questo repository sincronizzato con l'API Open Data di Normattiva, oltre al registro utilizzato per il controllo di integrità (*verify gate*).

## Utilizzo

    ./corpus-build --changed-only --output . \
      --registry tools/registry/must_have.json \
      --verify --cache-dir /tmp/cache --commit-msg-file /tmp/commit-msg.txt

`corpus-build` supporta due modalità di funzionamento:

- **Completa (default)** — rigenera da zero tutte le collezioni richieste.
- **`--changed-only`** — aggiornamento incrementale. Le collezioni per cui i campi `dataCreazione` e il numero di atti di Normattiva non sono cambiati rispetto al file `manifest.json` già presente nel repository vengono saltate (nessun download). Le collezioni modificate vengono scaricate e confrontate tramite hash con il file `.entries.json` di ogni collezione (voce zip -> sha256 dell'estrazione grezza); gli atti nuovi o modificati vengono convertiti e salvati, gli atti rimossi dallo zip vengono eliminati e il file `manifest.json` viene aggiornato sul posto. Una collezione priva di file `.entries.json` viene inizializzata calcolando gli hash dello zip senza riconvertire nulla.

La generazione è deterministica: il campo `fetched_at` nel frontmatter corrisponde alla data dello snapshot della collezione, pertanto la riconversione di atti non modificati produce file identici byte per byte ed evita commit superflui.

I download supportano la ripresa dei trasferimenti interrotti (HTTP Range) e riprovano automaticamente in caso di errori di rete temporanei, consentendo di scaricare in modo affidabile anche collezioni di grandi dimensioni.

L'opzione `--zip-override DIR` (una cartella contenente i file `<Collezione>.zip`) esegue il processo sui file zip locali anziché tramite API (utilizzata per test e simulazioni).

L'opzione `--commit-msg-file` genera il messaggio di commit in italiano descrittivo delle modifiche apportate, utilizzato dal workflow per `git commit -F`.

Il registro in `tools/registry/must_have.json` funge da gate di verifica (*verify gate*): il workflow annulla il push (ed esce con codice non zero) nel caso in cui un atto fondamentale sia mancante o fuori dall'intervallo previsto di articoli.

