# sed example

    sed -ri 's/(env PORT=)[[:digit:]]+/\15123/' /etc/init/fp-web.conf

    sed -r 's!(env DEFAULT_CENTER=).*$!\1"15/-17.8086/30.9282"!' fp-web.upstart