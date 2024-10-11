# Лаба 3 со звездочкой

В качестве секретохранилки использовалась Hashicorp Vault, но на Ubuntu поработать с ней не удалось ввиду того, что в России Hashicorp запрещен. Упорными действиями получилось добыть доступ к нему только на Windows.

## Настройка Vault.

После установки перейдем в консоль и настроим Hashicorp. При помощи `vault server -dev` переходим в режим разработки локально.

![2024-10-11_16-38-17](https://github.com/user-attachments/assets/4ab664dd-a580-4a3c-9dda-62739d802605)

Здесь мы увидим токен для аутентификации, заносим его в secrets. Здесь же указан url для подключения.

Теперь в режиме разработки задаем переменные окружения: адрес и токен.

![2024-10-11_18-44-34](https://github.com/user-attachments/assets/48b4c52e-c8fe-4796-a848-fab5227813a6)

Установим секреты при помощи `vault kv put`.

![2024-10-11_18-51-10](https://github.com/user-attachments/assets/abf39c48-1af9-4d6d-81e5-e46a8e71cdfa)

Созданы два секрета.

## Разберем пайплайн.

Первым делом импортируем секреты из Hashicorp. Указываем url для подключения к Vault, при помощи токена происходит аутентификация. Далее указываем секреты для импорта - `firstKey` и `secondKey`.

```
- name: Import Secrets
        id: import-secrets
        uses: hashicorp/vault-action@v2
        with:
        url: http://127.0.0.1:8200
        token: ${{ secrets.VAULT_TOKEN }}
        secrets: |
          secret/data/ci/keys firstKey | FIRST_SECRET_KEY ;
          secret/data/ci/keys secondKey | SECOND_SECRET_KEY ;
```

Теперь они будут доступны для использования в переменных `FIRST_SECRET_KEY` и `SECOND_SECRET_KEY`. Эти переменные можно использовать в других шагах.
