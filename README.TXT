***Alteração - RDM 334150 - Início - Contagem OP sem movimentação
  DATA: lt_pp_ef TYPE ty_t_ef_pp,
        ls_pp_ef TYPE ty_ef_pp.

  IF lt_chaves_mw[] IS NOT INITIAL.

    SELECT b~znumpp AS aufnr, b~znumoa, b~zmatat AS matnr, a~objnr
      FROM zrnppt_popa AS b
      INNER JOIN aufk AS a ON a~aufnr = b~znumpp
      FOR ALL ENTRIES IN @lt_chaves_mw
      WHERE b~zmatat = @lt_chaves_mw-matnr
      INTO CORRESPONDING FIELDS OF TABLE @lt_pp_ef.

    PERFORM f_ef_filtra_pp CHANGING lt_pp_ef.

    LOOP AT lt_pp_ef INTO ls_pp_ef.
      LOOP AT lt_chaves_mw INTO ls_chave_mw WHERE matnr = ls_pp_ef-matnr.
        CLEAR gs_ef_op_sem_mov.
        gs_ef_op_sem_mov-matnr = ls_chave_mw-matnr.
        gs_ef_op_sem_mov-werks = ls_chave_mw-werks.
        gs_ef_op_sem_mov-qtd   = 1.
        COLLECT gs_ef_op_sem_mov INTO gt_ef_op_sem_mov.
      ENDLOOP.
    ENDLOOP.

  ENDIF.
***Alteração - RDM 334150 - Fim
