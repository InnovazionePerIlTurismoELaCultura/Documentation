Gitolite: private repositories administration
=============================================

Gitolite viene usato per l'hosting remoto di repositories git.  Uitlizza SSH per autenticare gli utenti attraverso un meccansimo di autenticazione a chiave pubblica. 

#0. Installazione

Se siete interessati all'installazione potete dare un'occhiata al paragrafo 3 dei riferimenti.

#1. Prima configurazione
L'installazione prevede la creazione di un utente "gitolite" nel sistema linux (su CentOS l'utente è *gitolite*, se l'installazione è avvenuta manualmente, probabilmente l'avrete chiamato *git*). Su CentOS, la home dell'utente *gitolite* è accessibile al seguente path:

    /var/lib/gitolite

Li troverete tutto quello che vi serve per iniziare.

##1.1 Creazione delle chiavi SSH per l'amministrazione.
La gestione degli utenti e degli accessi ai repository non avviene modificando localmente i file, ma attraverso un repository di amministrazione. Per poter clonare in locale il repository di amministrazione è necessario creare le chiavi RSA per l'utente che amministrerà il sistema in quanto l'accesso ai repository (attraverso i comandi *clone, push, pull, etc.*) è possibile **solo attraverso l'autenticazione a chiave pubblica**. 

**Gitolite non supporta l'autenticazione con password (in realtà non è proprio così, ma la sua configurazione non è semplicissima).**

Per la creazione delle chiavi di amministrazione dei repository, andare sulla propria macchina (quella con cui amministrerete Gitolite) ed eseguite:

    ssh-keygen

Se avete una macchina windows, dovrete installare **msysGit**  (scaricabile da [qui](http://msysgit.github.io/)) e utilizzare **Git Bash**.

Una volta create le chiavi:

    cd ~/.ssh/
    cat id_rsa.pub

Il risultato dovrebbe essere una cosa del genere:
    
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB [...] redirect@redirect-Aspire-E1-571

Questa è la vostra chiave pubblica: copiatela nella clipboard o salvatela in un file... servirà a Gitolite per confermare la nostra identità.
Ora tornate sulla macchina server dove avete installato Gitolite e da root eseguite:

    [root@localhost ~] passwd gitolite
    Inserire nuova password UNIX: (inserite una password per l'utente gitolite)
    Reinserire la nuova password UNIX: (reinserite la password per conferma)
    [root@localhost ~] su gitolite
    [gitolite@localhost /root] cd ~
    [gitolite@localhost ~] gl-setup -q admin.pub

dove *admin.pub* è un file con all'interno la vostra chiave pubblica precedentemente creata. Tenete traccia del nome del file, dato che il nome che darete al file servirà per identificare il vostro utente nel file di configurazione. 

Usate secure copy o un editor di testo per creare il file sul server remoto 

*gl-setup* è un comando presente nel package di gitolite quindi, se lo avete installato a mano, dovrete specificare il path completo di dove l'avete installato. 

##1.2 Clonazione del repository di amministrazione

Una volta effettuato il setup dell'utente amministratore, tornare sulla propra macchina e digitare:

    git clone gitolite@ipaddress:gitolite-admin.git

se tutto è andato bene, git dovrebbe effettuare la copia del repository di amministrazione sulla propria macchina nella directory corrente.

#2. Gestire le autorizzazioni

Dopo la clonazione:

    cd gitolite-admin

nella directory sono presenti due sotto-directory: *"/conf"* e *"/keydir"*:

- Nella cartella */conf* è presente un unico file *gitolite.conf* che è il file di configurazione di gitolite. 
- Nella cartella */keydir* ci sono invece le chiavi pubbliche degli utenti che hanno l'accesso ai repository di gitolite.

##2.1 Creazione dei repository remoti
Per creare un repository sulla macchina remota, effettuare queste operazioni sulla macchina locale (dove è stato clonato il repository di amministrazione):

- Aprire il file *gitolite.conf* con il proprio editor preferito che appararià più o meno così:

        repo    gitolite-admin
                RW+     =   admin
        repo    testing
                RW+     =   @all

- Assumendo che vogliamo creare il repository *"backend"* modifichiamo il file in questo modo:

        repo    gitolite-admin
                RW+     =   admin
        repo    testing
                RW+     =   @all
        repo    backend.git
                RW+     =   admin

- In questo modo abbiamo specificato che gitolite dovrà creare un repository con nome *"backend"* che avrà un'unico utente chiamato *admin* che avrà accesso completo al repository (lettura,scrittura).

- Salviamo il file e, sempre in locale, diamo i seguenti comandi:

        [user@localhost ~/gitolite-admin] git add -A
        [user@localhost ~/gitolite-admin] git commit -m "Creato repository backend"
        [user@localhost ~/gitolite-admin] git push origin master

Una volta fatto il commit sul repository remoto, gitolite ci confermerà la creazione del repository e non resta che clonarlo ed effettuare il primo commit.

##2.2 Gestione dei permessi degli utenti

Supponiamo di voler dare l'accesso in sola lettura al repository *"backend"* all'utente *luca*:

- Chiediamo a Luca di generare la chiave RSA (così com abbiamo fatto per l'utente *admin*) e di inviarci la parte publica (in genere il file *id_rsa.pub*)
- Una volta ottenuta la chiave, copiamo il file sulla macchina dove abbiamo clonato il repository di amministrazione, più precisamente nella cartella:

        ~/gitolite-admin/keydir

- considerando che il nome che daremo al file servirà ad indentificare l'utente nel file di configurazione, rinominiamo il file in questo modo:

        [user@localhost ~/gitolite-admin/keydir] mv id_rsa.pub luca.pub

- Modifichiamo il file *gitolite.conf* in questo modo:

        repo    gitolite-admin
                RW+     =   admin
        repo    testing
                RW+     =   @all
        repo    backend.git
                RW+     =   admin
                R       =   luca

- Facciamo il commit dei cambiamenti ed effettuiamo il push sul server che hosta i repository:

        [user@localhost ~/gitolite-admin] git add -A
        [user@localhost ~/gitolite-admin] git commit -m "Aggiunto utente luca al repository backend in sola lettura"
        [user@localhost ~/gitolite-admin] git push origin master

- da questo momento in poi, Luca potrà clonare il repository in locale in questo modo:

        [luca@lucamachine] git clone gitolite@ipaddress:backend.git

     dove *"ipaddress"* è l'indirizzo ip (o l'hostname) della macchina dove gira gitolite.

#3 Riferimenti tecnici

[Gitolite documentation](http://www.google.com)

[Installazione e configurazione su CentoOS 6.4](http://sachinsharm.wordpress.com/2013/10/04/installsetup-and-configure-git-server-with-gitolite-and-gitweb-on-centosrhel-6-4/)
