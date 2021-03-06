# Типы хранилища


{{ mkf-name }} позволяет использовать для кластеров сетевое и локальное хранилища. Сетевое хранилище реализовано на базе сетевых блоков — виртуальных дисков в инфраструктуре {{ yandex-cloud }}. Локальное хранилище организуется на дисках, которые физически размещаются в [серверах-брокерах](brokers.md).

{% include [mkf-storage-type](../../_includes/mdb/storage-type.md) %}

## Особенности локального хранилища {#local-storage-features}

Локальное хранилище в кластере из одного хоста не обеспечивает отказоустойчивости: при отказе локального диска данные теряются безвозвратно. Чтобы обеспечить отказоустойчивость, создавайте кластеры из двух и более хостов.