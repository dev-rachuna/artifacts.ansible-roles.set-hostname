# <img src=".gitlab/ansible.png" alt="linux" height="20"/> set-hostname

Rola Ansible do ustawiania hostname'u serwera, aktualizacji wpisu 127.0.1.1 w `/etc/hosts`. Obsługuje systemy Debian, Ubuntu, AlmaLinux (EL 7/8/9) oraz Alpine.

---

## Co robi rola

- ustawia hostname na wartość FQDN lub krótki hostname (moduł `hostname`)
- aktualizuje wpis `127.0.1.1` w `/etc/hosts` — gdy nazwa zawiera kropkę, FQDN jest wpisywany jako pierwszy (zgodnie z konwencją `man hosts`), a krótki hostname jako alias
- opcjonalnie wymusza restart maszyny po zmianie (można wyłączyć)

---

## Architektura roli set-hostname

### Flow procesu

```mermaid
flowchart TD
    START([Start roli]) --> T1

    T1["<b>Change hostname</b><br/>ansible.builtin.hostname<br/>name: in_hostname_fqdn"]
    T1 -->|changed| N1([notify: Reboot system])
    T1 -->|ok| T2

    N1 --> T2

    T2["<b>Change /etc/hosts</b><br/>ansible.builtin.lineinfile<br/>regexp: ^127.0.1.1"]
    T2 -->|changed| N2([notify: Reboot system])
    T2 -->|ok| H_CHECK

    N2 --> H_CHECK

    H_CHECK{Czy handler<br/>został wywołany?}
    H_CHECK -->|Nie| DONE([Koniec roli])
    H_CHECK -->|Tak| COND1

    COND1{ansible_host<br/>!= localhost/127.0.0.1?}
    COND1 -->|Nie| SKIP([Pominięto reboot])
    COND1 -->|Tak| COND2

    COND2{in_disable_reboot<br/>== false?}
    COND2 -->|Nie| SKIP
    COND2 -->|Tak| REBOOT

    REBOOT["<b>Reboot system</b><br/>ansible.builtin.reboot<br/>timeout: 600s"]
    REBOOT --> DONE

    SKIP --> DONE
```

### Opis kroków

| Krok | Moduł | Opis |
|------|-------|------|
| Change hostname | `ansible.builtin.hostname` | Ustawia hostname na wartość `in_hostname_fqdn` |
| Change /etc/hosts | `ansible.builtin.lineinfile` | Aktualizuje wpis `127.0.1.1` — dla FQDN wpisuje pełną nazwę + alias krótki |
| Reboot system | `ansible.builtin.reboot` | Restartuje maszynę (timeout 600s), pomijany dla localhost i gdy `in_disable_reboot: true` |


## Wymagania

- Ansible 2.9+
- dostęp z uprawnieniami root (np. `become: true`)

---

## Zmienne

| Zmienna                          | Domyślna wartość           | Opis |
|----------------------------------|----------------------------|------|
| `in_hostname_fqdn`               | `{{ inventory_hostname }}` | Docelowy hostname/FQDN. Gdy zawiera kropkę, wpis w `/etc/hosts` zawiera hostname i FQDN. |
| `in_disable_reboot`              | `false`                    | `true` wyłącza reboot po zmianie hostname'u lub wpisu w `/etc/hosts`. |

---

## Użycie

Podstawowy playbook ustawiający hostname i wpis w `/etc/hosts`:

```yaml
- hosts: all
  become: true
  roles:
    - role: set-hostname
      vars:
        in_hostname_fqdn: app01.example.com
        in_disable_reboot: true  # pominie restart po zmianie
```

Zmiana hostname'u i wpisu w `/etc/hosts` wywołuje handler `Reboot system`. Jeśli chcesz uniknąć restartu (np. w środowisku testowym), ustaw `in_disable_reboot: true`.

---

## Contributions

Jeśli masz pomysły na ulepszenia, zgłoś problemy, rozwidl repozytorium lub utwórz Merge Request. Wszystkie wkłady są mile widziane!
[Contributions](CONTRIBUTING.md)

---

## License

[Licencja](LICENSE) oparta na zasadach Creative Commons BY-NC-SA 4.0, dostosowana do potrzeb projektu.

---

## Author Information

| ![Maciej Rachuna](https://gitlab.com/uploads/-/system/user/avatar/8161705/avatar.png?width=50) |
|------------------------------------------------------------------------------------------------|
| [Maciej Rachuna](https://gitlab.commrachuna)                                                   |
