***Alteração - RDM 334150 - Início - Seleção OP sem movimentação
            CLEAR: gt_ef_op_det[], gt_fieldcat[], lt_excl[], ls_excl.
            READ TABLE t_saida INDEX rs_selfield-tabindex.

            DATA lt_pp_det TYPE ty_t_ef_pp.
            CLEAR lt_pp_det.

            SELECT b~znumpp AS aufnr, b~znumoa, b~zmatat AS matnr,
                   a~objnr, a~erdat, c~gamng
              FROM aufk AS a
              INNER JOIN zrnppt_popa AS b ON b~znumpp = a~aufnr
              LEFT OUTER JOIN afko AS c ON c~aufnr = a~aufnr
              WHERE b~zmatat = @t_saida-matnr
              INTO CORRESPONDING FIELDS OF TABLE @lt_pp_det.

            " Mesma regra da contagem
            PERFORM f_ef_filtra_pp CHANGING lt_pp_det.

            LOOP AT lt_pp_det INTO DATA(ls_pp_det).
              CLEAR gs_ef_op_det.
              gs_ef_op_det-aufnr = ls_pp_det-aufnr.
              gs_ef_op_det-matnr = ls_pp_det-matnr.
              gs_ef_op_det-qtd   = ls_pp_det-gamng.
              gs_ef_op_det-erdat = ls_pp_det-erdat.
              gs_ef_op_det-objnr = ls_pp_det-objnr.
              APPEND gs_ef_op_det TO gt_ef_op_det.
            ENDLOOP.
***Alteração - RDM 334150 - Fim
