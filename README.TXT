FORM f_op_det_user_command USING r_ucomm    TYPE sy-ucomm
                                 rs_selfield TYPE slis_selfield.

  DATA ls_opt TYPE ctu_params.   "RDM 334150

  CASE r_ucomm.
    WHEN '&IC1'.  "Hotspot / duplo-clique no AUFNR → CO03
      READ TABLE gt_ef_op_det INTO gs_ef_op_det INDEX rs_selfield-tabindex.
      IF sy-subrc = 0 AND rs_selfield-fieldname = 'AUFNR'.
        SET PARAMETER ID 'ANR' FIELD gs_ef_op_det-aufnr.
        CALL TRANSACTION 'CO03' AND SKIP FIRST SCREEN.
      ENDIF.

***Alteração - RDM 334150 - Início - ZAGRUPA/ZDPR com os dados da linha
    WHEN 'ZAGRUPA'.  "Botão → ZAGRUPA (cancelamento total da OP)
      READ TABLE gt_ef_op_det INTO gs_ef_op_det INDEX rs_selfield-tabindex.
      IF sy-subrc <> 0.
        MESSAGE 'Posicione o cursor na linha da OP desejada' TYPE 'I'.
        RETURN.
      ENDIF.

      FREE t_bdc.
      PERFORM insere_bdc USING 'X' 'PROGRAMA_ZAGRUPA'  'TELA_ZAGRUPA'.
      PERFORM insere_bdc USING ''  'CAMPO_ORDEM_AGRUP' gs_ef_op_det-znumoa.
      PERFORM insere_bdc USING ''  'RADIO_CANC_TOTAL'  'X'.
      PERFORM insere_bdc USING ''  'CAMPO_CENTRO'      gs_ef_op_det-werks.

      CLEAR ls_opt.
      ls_opt-dismode = 'E'.   "Abre a tela já preenchida
      ls_opt-nobiend = 'X'.   "O usuário confere e executa
      CALL TRANSACTION 'ZAGRUPA' USING t_bdc OPTIONS FROM ls_opt.

    WHEN 'ZDPR'.  "Botão → ZDPR
      READ TABLE gt_ef_op_det INTO gs_ef_op_det INDEX rs_selfield-tabindex.
      IF sy-subrc <> 0.
        MESSAGE 'Posicione o cursor na linha da OP desejada' TYPE 'I'.
        RETURN.
      ENDIF.

      FREE t_bdc.
      PERFORM insere_bdc USING 'X' 'PROGRAMA_ZDPR'     'TELA_ZDPR'.
      PERFORM insere_bdc USING ''  'CAMPO_DIARIO'      gs_ef_op_det-diario.
      PERFORM insere_bdc USING ''  'CAMPO_CENTRO_DPR'  gs_ef_op_det-werks.

      CLEAR ls_opt.
      ls_opt-dismode = 'E'.
      ls_opt-nobiend = 'X'.
      CALL TRANSACTION 'ZDPR' USING t_bdc OPTIONS FROM ls_opt.
***Alteração - RDM 334150 - Fim

  ENDCASE.
ENDFORM.
