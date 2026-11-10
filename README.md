***Inserção - RDM 334150 - Início - Regra única "OP sem movimentação"
FORM f_ef_filtra_pp CHANGING ct_pp TYPE ty_t_ef_pp.

  TYPES: BEGIN OF ty_oa,
           znumoa TYPE zrnppt_popa-znumoa,
         END OF ty_oa,
         BEGIN OF ty_pp,
           aufnr TYPE aufk-aufnr,
         END OF ty_pp,
         BEGIN OF ty_st,
           objnr TYPE jest-objnr,
           stat  TYPE jest-stat,
         END OF ty_st.

  DATA: lt_oa_canc TYPE SORTED TABLE OF ty_oa WITH NON-UNIQUE KEY znumoa,
        lt_pconf   TYPE SORTED TABLE OF ty_pp WITH NON-UNIQUE KEY aufnr,
        lt_mov     TYPE SORTED TABLE OF ty_pp WITH NON-UNIQUE KEY aufnr,
        lt_stat    TYPE SORTED TABLE OF ty_st WITH NON-UNIQUE KEY objnr stat.

  CHECK ct_pp IS NOT INITIAL.

  " a) OP agrupadora cancelada (ZRNPPT_PHS) - mesmo critério do FORM SEL_ORD_PROD
  SELECT znumoa
    FROM zrnppt_phs
    FOR ALL ENTRIES IN @ct_pp
    WHERE znumoa  = @ct_pp-znumoa
      AND zstatus = 'CANC'
    INTO TABLE @lt_oa_canc.

  LOOP AT ct_pp ASSIGNING FIELD-SYMBOL(<ls_pp>).
    IF line_exists( lt_oa_canc[ znumoa = <ls_pp>-znumoa ] ).
      <ls_pp>-excluir = abap_true.
    ENDIF.
  ENDLOOP.
  DELETE ct_pp WHERE excluir = abap_true.

  " b) Uma linha por PP (a ZRNPPT_POPA tem várias linhas por PP)
  SORT ct_pp BY aufnr.
  DELETE ADJACENT DUPLICATES FROM ct_pp COMPARING aufnr.
  CHECK ct_pp IS NOT INITIAL.

  " c) Status: precisa ABER (I0001) ou LIB (I0002) e não pode FORN (I0012)
  SELECT objnr, stat
    FROM jest
    FOR ALL ENTRIES IN @ct_pp
    WHERE objnr = @ct_pp-objnr
      AND stat  IN ( 'I0001', 'I0002', 'I0012' )
      AND inact = @space
    INTO TABLE @lt_stat.

  " d) Registro na ZRNPPT_PCONFIT
  SELECT znumpp
    FROM zrnppt_pconfit
    FOR ALL ENTRIES IN @ct_pp
    WHERE znumpp = @ct_pp-aufnr
    INTO TABLE @lt_pconf.

  " e) Movimento na MSEG (regra do teste: 101, 261, 961, 531, 532, 962)
  SELECT aufnr
    FROM mseg
    FOR ALL ENTRIES IN @ct_pp
    WHERE aufnr = @ct_pp-aufnr
      AND bwart IN ( '101', '261', '961', '531', '532', '962' )
    INTO TABLE @lt_mov.

  LOOP AT ct_pp ASSIGNING <ls_pp>.
    IF NOT ( line_exists( lt_stat[ objnr = <ls_pp>-objnr stat = 'I0001' ] )
          OR line_exists( lt_stat[ objnr = <ls_pp>-objnr stat = 'I0002' ] ) )
       OR line_exists( lt_stat[ objnr = <ls_pp>-objnr stat = 'I0012' ] )
       OR line_exists( lt_pconf[ aufnr = <ls_pp>-aufnr ] )
       OR line_exists( lt_mov[ aufnr = <ls_pp>-aufnr ] ).
      <ls_pp>-excluir = abap_true.
    ENDIF.
  ENDLOOP.
  DELETE ct_pp WHERE excluir = abap_true.

ENDFORM.
***Inserção - RDM 334150 - Fim
