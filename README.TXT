 DELETE ADJACENT DUPLICATES FROM lt_chaves COMPARING matnr.

  SELECT zmatat znumpp
    INTO TABLE lt_popa
    FROM zrnppt_popa
    FOR ALL ENTRIES IN lt_chaves_mw
    WHERE zmatat = lt_chaves_mw-matnr.

  DATA(lt_popa_aux) = lt_popa[].
  SORT lt_popa_aux BY znumpp.
  DELETE ADJACENT DUPLICATES FROM lt_popa_aux COMPARING znumpp.

  IF lt_popa[] IS NOT INITIAL.
    SELECT aufnr objnr
      INTO TABLE lt_aufk
      FROM aufk
      FOR ALL ENTRIES IN lt_popa_aux
      WHERE aufnr = lt_popa_aux-znumpp.

    IF lt_aufk[] IS NOT INITIAL.
      SELECT objnr stat inact
        INTO TABLE lt_jest
        FROM jest
        FOR ALL ENTRIES IN lt_aufk
        WHERE objnr = lt_aufk-objnr
          AND ( stat = 'I0001' OR stat = 'I0002' )
          AND inact = ''.
    ENDIF.

    SELECT znumpp
      INTO TABLE lt_pconf
      FROM zrnppt_pconfit
      FOR ALL ENTRIES IN lt_popa_aux
      WHERE znumpp = lt_popa_aux-znumpp.

    SORT lt_aufk BY aufnr.
    SORT lt_jest BY objnr stat.
    SORT lt_pconf BY znumpp.

    LOOP AT lt_popa INTO ls_popa.
      CLEAR: lv_has_aber_lib, lv_has_conf.

      READ TABLE lt_aufk INTO ls_aufk WITH KEY aufnr = ls_popa-znumpp BINARY SEARCH.
      IF sy-subrc <> 0.
        CONTINUE.
      ENDIF.

      READ TABLE lt_jest INTO ls_jest WITH KEY objnr = ls_aufk-objnr stat = 'I0001' BINARY SEARCH.
      IF sy-subrc = 0.
        lv_has_aber_lib = 'X'.
      ENDIF.

      READ TABLE lt_jest INTO ls_jest WITH KEY objnr = ls_aufk-objnr stat = 'I0002' BINARY SEARCH.
      IF sy-subrc = 0.
        lv_has_aber_lib = 'X'.
      ENDIF.

      READ TABLE lt_pconf INTO ls_pconf WITH KEY znumpp = ls_popa-znumpp BINARY SEARCH.
      IF sy-subrc = 0.
        lv_has_conf = 'X'.
      ENDIF.

      IF lv_has_aber_lib = 'X' AND lv_has_conf IS INITIAL.
        LOOP AT lt_chaves_mw INTO ls_chave_mw WHERE matnr = ls_popa-matnr.
          READ TABLE gt_ef_op_sem_mov INTO gs_ef_op_sem_mov WITH KEY matnr = ls_chave_mw-matnr
                                                                        werks = ls_chave_mw-werks.
          IF sy-subrc EQ 0.
            DATA(lv_tabix) = sy-tabix.
          ELSE.
            CLEAR: gs_ef_op_sem_mov, lv_tabix.
            gs_ef_op_sem_mov-matnr = ls_chave_mw-matnr.
            gs_ef_op_sem_mov-werks = ls_chave_mw-werks.
          ENDIF.
          gs_ef_op_sem_mov-qtd = gs_ef_op_sem_mov-qtd + 1.
          IF lv_tabix IS NOT INITIAL.
            MODIFY TABLE gt_ef_op_sem_mov FROM gs_ef_op_sem_mov.
          ELSE.
            INSERT gs_ef_op_sem_mov INTO TABLE gt_ef_op_sem_mov.
          ENDIF.
        ENDLOOP.
      ENDIF.
    ENDLOOP.
  ENDIF.
