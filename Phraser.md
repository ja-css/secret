Block structures for Phraser
-----------------------------------------------------
**Quick Facts**:
- AES encryption: CBC - most secure, ECB - least secure, CTR - in the middle.
	- see https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_codebook_(ECB)
- AES-256/192/128: key size 256/192/128 bits or 32/24/16 bytes
- Data doesn't multiply on AES encryption, i.e. a 64 byte long buffer encrypted is also 64 bytes.
- AES - text length must be evenly divisible by 16 bytes `(str_len % 16 == 0)`. That is 128 bit block size. You should pad the end of the string with zeros if this is not the case.
	- In our case all blocks are 4096 bytes; `4096 = 256*16`.
-----------------------------------------------------
Super-Structure: **Abstract Block**
- max 4'294'967'296 blocks (hw - max 384 at the same time)
- max 4'294'967'296 versions of each block
- max 256 block types
- checksum Adler16
- header - 9, footer - 2, total overhead - 11 bytes
```
|id_block|version|type|     |checksum|
|   4    |   4   |  1 |.....|    2   |
```
-----------------------------------------------------
**Folders Block**: Splitter/Spawner, id reuse
- 256 folders max, id reuse
- max folder name 256 bytes
- phrases sorted alphabetically inside a folder
```
|folder_count|
|     1      |
...|id_folder|id_parent|length_name|name|
...|    1    |    1    |     1     |  N |
```
-----------------------------------------------------
**Symbol Sets Block**: Splitter/Spawner, id reuse
- 256 sets max
- name max 256 bytes
- set max 256 bytes
```
|set_count|
|    1    |
...|id_set|length_name|name|length_set|set|
...|   1  |     1     |  N |    1     | M |
```
-----------------------------------------------------
Sub-Structure: **Phrase**
- max 256 words per phrase
- word flags: `generatable`/`inputable`/`viewable`/`typeable`/`starred`
	- `starred` is materialized only in **Phrase Block**
- max word name 256 bytes
- minimum and maximum length is between 1 and 65'536 (probably restrict practical max to 2'048 or 1'024)
- max 256 symbol sets per word
  - symbol_sets only exist if (`generatable`==true)
```
|word_count|
|    1     |
...|id_word|flags|min_length|max_length|serial|length_name|name|symbol_set_count|symbol_set_ids|
...|   1   |  1  |     2    |     2    |   1  |     1     |  N |       1        |       S      |
```
-----------------------------------------------------
**Phrase Templates Block**: Splitter/Spawner
- max 256 templates
- max template name 256 bytes
```
|template_count|
|       1      |
...|id_template|length_name|name|
...|     1     |     1     |  N |
......|PHRASE|
......|   P  |
```
-----------------------------------------------------
**Phrase Block**: Solid State
- `id_block` is `id_phrase`
- max phrase name 256 bytes
- max history count - 256 (highly unlikely to reach that value due to block size of 4'096)
- max words per history entry is 256, mirroring PHRASE structure
- max word length is 65'536, just to allow for crazy generated passwords
```
|id_folder|length_name|name|
|    1    |     1     |  N |
|PHRASE|
|   P  |
|history_count|
|      1      |
...|word_count|
...|    1     |
......|id_word|length_word|word|
......|   1   |     1     |  N |
```
-----------------------------------------------------
/* 
 *  Default templates, e.g.:
 *  
 *  Template - Simple
 *    - login
 *    - password
 *  
 *  Template - Website
 *    - url
 *    - login
 *    - password
 *  
 *  Template - Security Questions
 *    - security question 1
 *    - security answer 1
 *    - security question 2
 *    - security answer 2
 *    - security question 3
 *    - security answer 3
 *  
 *  Template - Trading
 *    - login
 *    - password
 *    - trading pin
 */
