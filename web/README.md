# Web frontend – dokumentace

Tato složka popisuje frontend (TypeScript/React) pro správu **Hekatoncheiros Core**. Dokumentace je psaná tak, aby šla použít jako **základ pro další aplikace**, zejména UI kit a komponenty.

## Cíle

- poskytnout základní ovládací rozhraní pro core/kernel
- definovat sdílené UI komponenty (tlačítka, formuláře, layout)
- popsat základní architekturu a datové toky

## Runtime závislosti

- Backend Core API: `http://127.0.0.1:3000`
- Database (PostgreSQL) běží v dockeru na portu `5432`

## Struktura dokumentace

- [Architektura frontendu](./architecture.md)
- [UI kit / Design systém](./ui-kit.md)
- [Ovládací rozhraní Core/Kernel](./core-kernel-console.md)

## Konvence

- Dokumentace preferuje **komponentový přístup** a **opakovatelné UI prvky**.
- Komponenty jsou navrženy tak, aby byly **použitelné i v jiných aplikacích**.
- URL rozhraní odpovídají core API surface (`/api/v1/...`).
