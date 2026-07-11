```
chezmoi docker ubuntu:22.04 --apply phdoanh
docker rm -f <container_id>

chezmoi init phdoanh          # chỉ clone, CHƯA apply
chezmoi diff                  # xem trước những gì sẽ bị ghi đè
chezmoi apply                 # chỉ chạy khi đã chấp nhận diff

encrypted_private_dot_zsh_secrets.age
```
