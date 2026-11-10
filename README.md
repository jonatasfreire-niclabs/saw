      SORT lt_ekpo BY matnr werks ebeln ebelp.        ← já existia, ficou

***Alteração - RDM 334150 - Início - qualquer registro na EKBE exclui o item
      SELECT DISTINCT ebeln, ebelp
        INTO TABLE @DATA(lt_ekbe_aux)
        ...
      ENDLOOP.
***Alteração - RDM 334150 - Fim

    ENDIF.                                            ← já existia (era a 18316), ficou

*    IF lt_ekpo[] IS NOT INITIAL.                     ← comentado, ficou
