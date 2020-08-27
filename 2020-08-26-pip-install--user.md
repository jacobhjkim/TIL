# PIP install Gotcha

While I was making a Dockerfile for Airflow, some python packages were installed under `/home`, while some Python packages are installed under `/usr/local/bin`. This was caused via using `--user` option in `pip install`.

All this time, I assumed `--user` option was some sort of `sudo` for `pip install`. Little did I know, `--user` makes pip install packages in your home directory instead, which doesn't require any special privileges. [source](https://stackoverflow.com/a/42989020)

🙃