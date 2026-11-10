          CLEAR: gt_ef_op_det[], gt_fieldcat[], lt_excl[], ls_excl.
            READ TABLE t_saida INDEX rs_selfield-tabindex.

            SELECT a~aufnr b~zmatat AS matnr c~gamng AS qtd a~erdat a~objnr
              INTO CORRESPONDING FIELDS OF TABLE gt_ef_op_det
              FROM aufk AS a
              INNER JOIN zrnppt_popa AS b ON b~znumpp = a~aufnr
              LEFT JOIN afko AS c ON c~aufnr = a~aufnr
             WHERE b~zmatat = t_saida-matnr.

            LOOP AT gt_ef_op_det INTO gs_ef_op_det.
              v_tab_aux = sy-tabix.

              SELECT SINGLE stat
                INTO lv_retorno
                FROM jest
               WHERE objnr = gs_ef_op_det-objnr
                 AND stat  = 'I0001'
                 AND inact = ''.

              IF lv_retorno IS INITIAL.
                SELECT SINGLE stat
                  INTO lv_retorno
                  FROM jest
                 WHERE objnr = gs_ef_op_det-objnr
                   AND stat  = 'I0002'
                   AND inact = ''.
              ENDIF.

              IF lv_retorno IS INITIAL.
                DELETE gt_ef_op_det INDEX v_tab_aux.
                CONTINUE.
              ENDIF.

              CLEAR lv_retorno.
              SELECT SINGLE stat
                INTO lv_retorno
                FROM jest
               WHERE objnr = gs_ef_op_det-objnr
                 AND stat  = 'I0012'
                 AND inact = ''.
              IF sy-subrc = 0.
                DELETE gt_ef_op_det INDEX v_tab_aux.
                CONTINUE.
              ENDIF.

              SELECT SINGLE znumpp
                INTO lv_retorno
                FROM zrnppt_pconfit
               WHERE znumpp = gs_ef_op_det-aufnr.
              IF sy-subrc = 0.
                DELETE gt_ef_op_det INDEX v_tab_aux.
                CONTINUE.
              ENDIF.
            ENDLOOP.
