      SORT lt_ekpo BY matnr werks ebeln ebelp.

***Alteração - RDM 334150 - Início - qualquer registro na EKBE exclui o item
      SELECT DISTINCT ebeln, ebelp
        INTO TABLE @DATA(lt_ekbe_aux)
        FROM ekbe
        FOR ALL ENTRIES IN @lt_ekpo
        WHERE ebeln EQ @lt_ekpo-ebeln
          AND ebelp EQ @lt_ekpo-ebelp.

      SORT lt_ekbe_aux BY ebeln ebelp.

      LOOP AT lt_ekpo INTO ls_ekpo.
        READ TABLE lt_ekbe_aux WITH KEY ebeln = ls_ekpo-ebeln
                                        ebelp = ls_ekpo-ebelp
                                        TRANSPORTING NO FIELDS
                                        BINARY SEARCH.
        IF sy-subrc EQ 0.
          CONTINUE.
        ENDIF.

        CLEAR: gs_ef_ped_sem_mov.
        gs_ef_ped_sem_mov-matnr = ls_ekpo-matnr.
        gs_ef_ped_sem_mov-werks = ls_ekpo-werks.
        gs_ef_ped_sem_mov-qtd   = 1.
        COLLECT gs_ef_ped_sem_mov INTO gt_ef_ped_sem_mov.
      ENDLOOP.
***Alteração - RDM 334150 - Fim

    ENDIF.
