# dimitri.hautot.be

CV en ligne de Dimitri, site statique généré avec Quarkus Roq (thème `quarkus-roq-theme-resume`), hébergé chez Hostinger (LiteSpeed).

## Commandes

- `./mvnw quarkus:dev` : prévisualisation sur http://localhost:8080/
- `QUARKUS_ROQ_GENERATOR_BATCH=true ./mvnw package quarkus:run -DskipTests` : génère le site statique dans `target/roq/` (c'est ce dossier qui est déployé)

Roq est en version `999-SNAPSHOT` : le fork de Dimitri, construit localement (`~/perso/quarkus-roq`).

## Structure

- `data/` : contenu du CV (`bio.yml`, `profile.yml`, `social.yml`)
- `content/` : pages (`index.html`, `404.html`)
- `public/` : fichiers copiés tels quels à la racine du site (images, `.htaccess`)
- `web/` : CSS propre au projet, intégré au bundle

## Mozilla Observatory

Objectif : garder la note la plus haute possible sur https://developer.mozilla.org/en-US/observatory. Les en-têtes de sécurité sont définis dans `public/.htaccess`, publié à la racine du site par Roq (vérifier sa présence dans `target/roq/` après génération). L'Observatory note chaque sous-domaine séparément : ce fichier ne vaut que pour dimitri.hautot.be.

### Exception connue : `'unsafe-eval'`

Le thème resume utilise Alpine.js, qui évalue ses directives (`x-data`, `x-show`, `@click`…) avec `new Function()`. La CSP autorise donc `'unsafe-eval'` dans `script-src`, ce qui coûte des points. Pour l'enlever : passer le thème au build CSP d'Alpine (`@alpinejs/csp`) et remplacer les expressions des directives par des composants déclarés avec `Alpine.data()`. C'est un changement à faire dans le fork de Roq, pas ici.

### Règles

- **Aucun JavaScript inline** : pas de `<script>` sans `src`, pas d'attribut `on…`, pas d'URL `javascript:` dans les templates ou le contenu. Les scripts passent par le bundle (`/static/bundle/`).
- Ne pas ajouter de nouvelles expressions Alpine qui exigeraient d'autres assouplissements. Ne jamais ajouter `'unsafe-inline'` à `script-src`.
- **Aucune ressource externe** : pas de CDN, Google Fonts (la police Inter est déjà embarquée dans le bundle), analytics ou widget tiers. Les logos et images vont dans `public/images/`. Les liens `<a href>` vers d'autres sites ne sont pas concernés.
- Les styles inline sont tolérés (`style-src 'unsafe-inline'`, non pénalisé ; Alpine en a besoin pour `x-show`).
- Toute nouvelle source légitime s'ajoute à la seule directive concernée (`connect-src`, `font-src`, `img-src`…), jamais à `default-src`. Une ressource externe inévitable se charge avec SRI (`integrity` + `crossorigin="anonymous"`).
- Conserver dans `public/.htaccess` : HSTS `max-age` ≥ 6 mois avec `includeSubDomains`, `nosniff`, `frame-ancestors 'none'` + `X-Frame-Options: DENY`, `Referrer-Policy`, `Permissions-Policy`, et le blocage des fichiers cachés (sauf `/.well-known/`).

### Vérifier

- Après génération : servir `target/roq/` avec la CSP de `public/.htaccess` et vérifier l'absence de violation CSP dans la console du navigateur (le mode sombre et les boutons « Show more » doivent fonctionner).
- Après déploiement : `curl -sI https://dimitri.hautot.be/`, puis relancer l'Observatory.

## Git

- Messages de commit en anglais, courts.
- Ne pas committer `.idea/`, `target/`, `node_modules/`. Ne pas pousser sans demande explicite.
