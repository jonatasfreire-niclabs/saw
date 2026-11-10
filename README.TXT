           DATA(lt_ped) = gt_ef_ped_sem_mov[].
            DELETE lt_ped WHERE matnr NE t_saida-matnr.

            IF lt_ped[] IS NOT INITIAL.

              " 1. Busca pedidos abertos: EKPO + EKKO (sem JOIN em j_1bnflin)
              SELECT DISTINCT a~ebeln a~ebelp a~matnr
                a~menge AS qtd
                b~bedat AS data
                a~banfn a~werks
                INTO CORRESPONDING FIELDS OF TABLE gt_ef_ped_det
                FROM ekpo AS a
                INNER JOIN ekko AS b ON b~ebeln = a~ebeln
               WHERE a~matnr = t_saida-matnr
                 AND a~werks = t_saida-werks
                 AND a~elikz NE abap_true "= abap_false
                 AND a~loekz EQ abap_false
                 AND a~pstyp EQ '3'
                 AND b~bstyp = 'F'.

              IF sy-subrc EQ 0.
                SORT gt_ef_ped_det BY ebeln ASCENDING
                                      ebelp ASCENDING.
              ENDIF.

              " 2. Remove pedidos que já tiveram movimentação de entrada (EKBE)
              DATA: lt_mseg TYPE TABLE OF ekbe,
                    ls_mseg TYPE ekbe.
              SELECT DISTINCT ebeln ebelp
                INTO CORRESPONDING FIELDS OF TABLE lt_mseg
                FROM mseg
                FOR ALL ENTRIES IN gt_ef_ped_det
                WHERE ebeln EQ gt_ef_ped_det-ebeln
                  AND ebelp EQ gt_ef_ped_det-ebelp.

              IF sy-subrc EQ 0.
                SORT lt_mseg BY ebeln ASCENDING
                                ebelp ASCENDING.
              ENDIF.

              SELECT DISTINCT ebeln, ebelp
                INTO TABLE @DATA(lt_ekbe)
                FROM ekbe
                FOR ALL ENTRIES IN @gt_ef_ped_det
                WHERE ebeln EQ @gt_ef_ped_det-ebeln
                  AND ebelp EQ @gt_ef_ped_det-ebelp
                  AND bwart IN ( '101', '543' )
                  AND elikz NE @abap_true.

              IF sy-subrc EQ 0.
                SORT lt_ekbe BY ebeln ASCENDING
                                ebelp ASCENDING.
              ENDIF.

              LOOP AT gt_ef_ped_det INTO DATA(ls_ekpo).

                DATA(lv_tabix) = sy-tabix.

                READ TABLE lt_mseg WITH KEY ebeln = ls_ekpo-ebeln
                                            ebelp = ls_ekpo-ebelp
                                            TRANSPORTING NO FIELDS
                                            BINARY SEARCH.
                IF sy-subrc EQ 0.
                  DELETE gt_ef_ped_det INDEX lv_tabix.
                  CONTINUE.
                ENDIF.

                READ TABLE lt_ekbe WITH KEY ebeln = ls_ekpo-ebeln
                                            ebelp = ls_ekpo-ebelp
                                            TRANSPORTING NO FIELDS
                                            BINARY SEARCH.
                IF sy-subrc EQ 0.
                  DELETE gt_ef_ped_det INDEX lv_tabix.
                ENDIF.

              ENDLOOP.
