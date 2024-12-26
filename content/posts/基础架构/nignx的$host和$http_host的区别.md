> I would like to add another important point not mentioned in the accepted answer.
> 
> \$host do NOT have port number, while $http_host DO include the port number.
> 
> edit: not always.
> 
> I set up a header "add_header Y-blog-http_host "$http_host" always;"
> 
> Then curl -I -L domain.com:80 (or 443) and the header doesn't show a port number at all. Verified with nginx-extra 1.10.3. Is is because it's common http(s) ports or nginx configuration? This comment just to say things do not always behave the way you think.
> 
> 
