***Inserção - RDM 334150 - Início
TYPES: BEGIN OF ty_ef_pp,
         aufnr   TYPE aufk-aufnr,
         znumoa  TYPE zrnppt_popa-znumoa,
         matnr   TYPE mara-matnr,
         objnr   TYPE aufk-objnr,
         erdat   TYPE aufk-erdat,
         gamng   TYPE afko-gamng,
         excluir TYPE c LENGTH 1,
       END OF ty_ef_pp,
       ty_t_ef_pp TYPE STANDARD TABLE OF ty_ef_pp WITH DEFAULT KEY.
***Inserção - RDM 334150 - Fim
