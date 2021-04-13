---
title: 'Aktywacja zgodności Healthcare (HDS)'
slug: aktywacja-hds
excerpt: "Dowiedz się, jak włączyć opcję hostingu danych medycznych w Twoim projekcie Public Cloud"
section: 'Zarządzanie projektami'
order: 9
---

> [!primary]
> Tłumaczenie zostało wygenerowane automatycznie przez system naszego partnera SYSTRAN. W niektórych przypadkach mogą wystąpić nieprecyzyjne sformułowania, na przykład w tłumaczeniu nazw przycisków lub szczegółów technicznych. W przypadku jakichkolwiek wątpliwości zalecamy zapoznanie się z angielską/francuską wersją przewodnika. Jeśli chcesz przyczynić się do ulepszenia tłumaczenia, kliknij przycisk „Zaproponuj zmianę” na tej stronie.
>

**Ostatnia aktualizacja z dnia 22-03-2021**

## Wprowadzenie

Aby infrastruktura i platformy Public Cloud spełniały wymagania [dotyczące hostingu danych medycznych](https://www.ovhcloud.com/pl/enterprise/certification-conformity/hds/) we Francji, dane te muszą zostać certyfikowane jako "HDS".

Usługi OVHcloud Public Cloud posiadają certyfikaty HDS dla niektórych działań wymienionych w [repozytorium certyfikacji](https://esante.gouv.fr/labels-certifications/hds/certification-des-hebergeurs-de-donnees-de-sante){.external} Agencji Cyfrowej ds. Zdrowia.

|Usługi Public Cloud|Działania z certyfikatem HDS|
|---|---|
|Compute, Storage (Block, Object, Archive, Snapshot, Instance Backup)|1,2,3,4,6|
|Managed Kubernetes Service, Cloud Database, Data Processing (Spark), AI Training, ML Serving|1,2,3,4,5,6|

> [!primary]
>
> Usługi Managed Private Registry i Big Data Cluster (Hadoop) nie są w danym momencie przypisane do certyfikatu HDS.
>

**Dowiedz się, jak włączyć opcję hostingu danych medycznych w Twoim projekcie Public Cloud**

## Wymagania początkowe

- Dostęp do [Panelu klienta OVHcloud](https://www.ovh.com/auth/?action=gotomanager&from=https://www.ovh.pl/&ovhSubsidiary=pl){.external}.
- Wykupienie [poziomu wsparcia Business lub Enterprise](https://www.ovhcloud.com/pl/support-levels/) na Twoim koncie OVHcloud

## W praktyce

### Włącz opcję HDS podczas tworzenia nowego projektu Public Cloud

Utwórz w [Panelu klienta](https://www.ovh.com/auth/?action=gotomanager&from=https://www.ovh.pl/&ovhSubsidiary=pl) nowy projekt Public Cloud.

Jeśli wykupiłeś poziom wsparcia Business lub Enterprise, możesz zaznaczyć pole `Hosting danych medycznych i certyfikat HDS dla tego projektu`.

Otrzymasz wówczas dostęp do szczegółowych warunków hostingu danych medycznych. Zapoznaj się z tymi ostatnimi i zaznacz odpowiednie pole. Kliknij `Dalej`{.action}, aby zakończyć tworzenie projektu. Usługi tego projektu będą certyfikowane przez HDS zgodnie z warunkami umowy.

![włączyć HDS nowego projektu](images/hds-new-project.png){.thumbnail}

### Włącz opcję HDS dla istniejącego projektu Public Cloud

W [Panelu klienta OVHcloud](https://www.ovh.com/auth/?action=gotomanager&from=https://www.ovh.pl/&ovhSubsidiary=pl) wybierz projekt Public Cloud, dla którego chcesz, aby usługi miały certyfikat HDS.

Kliknij `Settings`{.action}. Jeśli wykupiłeś poziom wsparcia Business lub Enterprise, możesz zaznaczyć pole `Hosting danych medycznych i certyfikat HDS dla tego projektu`.

Otrzymasz wówczas dostęp do szczegółowych warunków hostingu danych medycznych.

Zapoznaj się z tymi regulaminami i zaznacz kratkę akceptacji regulaminów. Następnie kliknij `Aktualizuj`{.action}, aby dokończyć proces certyfikacji HDS dla Twojego projektu zgodnie z warunkami określonymi w warunkach zamówienia.

![aktywuj istniejący projekt HDS](images/hds-current-project.png){.thumbnail}

## Sprawdź również

[Dokumentacja Public Cloud](../)

Dołącz do społeczności naszych użytkowników na stronie<https://community.ovh.com/en/>.
