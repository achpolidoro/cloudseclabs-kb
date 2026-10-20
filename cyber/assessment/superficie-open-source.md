# Assessment de superfície com ferramentas open source

> **Subcategoria:** `cyber/assessment/`

```bash
subfinder -d exemplo-lab.com.br -silent -o subdominios.txt
httpx -l subdominios.txt -title -status-code -tech-detect
```
