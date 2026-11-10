***Alteração - RDM 334150 - Início
            DATA(lt_ped) = gt_ef_ped_sem_mov[].
            DELETE lt_ped WHERE matnr NE t_saida-matnr
                             OR werks NE t_saida-werks.

            IF lt_ped[] IS NOT INITIAL.

              SELECT DISTINCT a~ebeln a~ebelp a~matnr
                a~menge AS qtd
                b~bedat AS data
                a~banfn a~werks
                INTO CORRESPONDING FIELDS OF TABLE gt_ef_ped_det
                FROM ekpo AS a
                INNER JOIN ekko AS b ON b~ebeln = a~ebeln
               WHERE a~matnr = t_saida-matnr
                 AND a~werks = t_saida-werks
                 AND a~elikz NE abap_true
                 AND a~loekz EQ abap_false
                 AND a~pstyp EQ '3'
                 AND b~bstyp = 'F'.

              IF gt_ef_ped_det[] IS NOT INITIAL.   "Proteção do FOR ALL ENTRIES

                SORT gt_ef_ped_det BY ebeln ebelp.

                SELECT DISTINCT ebeln, ebelp
                  INTO TABLE @DATA(lt_ekbe)
                  FROM ekbe
                  FOR ALL ENTRIES IN @gt_ef_ped_det
                  WHERE ebeln EQ @gt_ef_ped_det-ebeln
                    AND ebelp EQ @gt_ef_ped_det-ebelp.

                SORT lt_ekbe BY ebeln ebelp.

                LOOP AT gt_ef_ped_det INTO DATA(ls_ekpo).
                  DATA(lv_tabix) = sy-tabix.
                  READ TABLE lt_ekbe WITH KEY ebeln = ls_ekpo-ebeln
                                              ebelp = ls_ekpo-ebelp
                                              TRANSPORTING NO FIELDS
                                              BINARY SEARCH.
                  IF sy-subrc EQ 0.
                    DELETE gt_ef_ped_det INDEX lv_tabix.
                  ENDIF.
                ENDLOOP.

              ENDIF.
***Alteração - RDM 334150 - Fim
