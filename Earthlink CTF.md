# Earthlink CTF
## Web Category
Challange Name : [ Bi Database Analyst ]
Poinst : 80
Category : WEB
#### Challenge Description
The challenge provided a login page that authenticated users via cookies. Upon analyzing the application, it was found that manipulating the cookie value granted access to a page with SQL query execution capabilities. The goal was to retrieve the contents of a `.txt` file stored on the server.

**Exploiting the Vulnerability:**
Used the following SQL command to retrieve the contents of the desired text file:

   `RUNSCRIPT FROM 'file:/tmp/flag.txt';`

The server executed the command and returned the content of the file, which included the flag. 

> `CTF{HERE_U_GO_H2_JDBC_ATTACK}`

## Misc Category
Challange Name : [  The Power Of cloud DeveOps ]
Poinst : 100
Category : Misc

#### Challenge Description
The challenge provided the following Azure Storage Blob URL:

`https://earthlinkctf.blob.core.windows.net/$web/`

> The URL pointed to the `$web` container in the Azure Storage Blob
> service, commonly used to host static websites.

If a file had been removed, Azure Blob storage may retain versions or metadata depending on the configuration.
Tested the URL for versioned blobs by appending query parameters like `?comp=versions`
By Use Azure Storage Explorer to connect to a blob anonymously.
Following This Parameters Retrun list of file 
`?restype=container&comp=list`



