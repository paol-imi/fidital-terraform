# S3

**TODO:** Nomenclature da rivedere

## Buckets

- Bucket per l’import:
  - `fidi-doc-import-demo`
  - `fidi-doc-import-acarpia`
  - `fidi-doc-import-fidital`

**TODO:** Controlla permessi in generale

## Policies

- Policy per accesso a bucket:
  - `fidi-doc-import-demo-s3-access`
  - `fidi-doc-import-acarpia-s3-access`
  - `fidi-doc-import-fidital-s3-access`

**TODO:** Diminuisci permessi

## Roles

- Ruoli da assegnare ai task ECS:
  - `role-import-demo` {policy.fidi-doc-import-demo-s3-access}
  - `role-import-acarpia` {fidi-doc-import-acarpia-s3-access}
  - `role-import-fidital` {fidi-doc-import-fidital-s3-access}

**TODO:** Definisci gli ID dei task che possono usare il ruolo
