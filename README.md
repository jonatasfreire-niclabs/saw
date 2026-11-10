************************************************************************
* PROGRAMA: ZRNPPR_PLANEJ_PROD                                         *
* DESCRIÇÃO: Planejamento de Produção                                  *
*                                                                      *
* AUTORES: Leonardo Bruno de Freitas Tércio                            *
* DATA: 11/12/2007                                                     *
************************************************************************
************************************************************************
*-----------------------------------------------------------------------
* Alteração - Jean Abraão Ferreira - Req.:RDRK925018
* SONDAPROCWORK
*-----------------------------------------------------------------------
*----------------------------------------------------------------------*
* Autor    : Grazielle C. Alves - SONDA PROCWORK      Data: 12.09.2008 *
* Request  : RDRK926403                                                *
* Descrição: Acerto na quebra por Máquina/ Acerto nas OV's e Relatório *
*            por Material                                              *
*----------------------------------------------------------------------*
************************************************************************
"REPORT zrnppr_planej_prod MESSAGE-ID zmenrlt.
"TYPE-POOLS: icon.

INCLUDE: zrnppr_planej_prod_top, zrnppr_planej_prod_form.

*----------------------------------------------------------------------*
*       CLASS lcl_dep DEFINITION
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
CLASS lcl_dep DEFINITION.

  PUBLIC SECTION.

    METHODS: gera_arquivo_serv.

    CLASS-METHODS: f4 RETURNING VALUE(file) TYPE rlgrap-filename.

  PRIVATE SECTION.

ENDCLASS.                    "lcl_
************************************************************************
**  Definição de Tabelas do Dicionário de Dados                       **
************************************************************************
*TABLES: crhd, vbep, t001w, vbap, aufk,
*        afvc, vbup, zrnppt_contrprog, caufv,
*        zrnppt_dpr, zrnppt_popa, mast, stpo, zrnppt_ov_separa,
*        crvm_b, mdpb, kna1, zrnppt_prev, zqmt_partnumber.
*----------------------------------------------------------------------*
* Tipos fundamentais para saída de um relatório ALV e cabeçalho
*----------------------------------------------------------------------*
TYPE-POOLS: slis, kkblo.
*----------------------------------------------------------------------*
* Parâmetros gerais para as funções do ALV
*----------------------------------------------------------------------*
* Parâmetros gerais do layout do relatório
DATA: gs_layout   TYPE slis_layout_alv.                " Layout de saída
DATA: gt_sort     TYPE slis_sortinfo_alv OCCURS 0 WITH HEADER LINE.
* Auxiliares na montagem do lay-out do relatório na funçao do "ALV".
DATA: gs_header TYPE kkblo_listheader,
      gt_header TYPE kkblo_t_listheader.

* Para o catálogo de campos
DATA: gt_fieldcat TYPE slis_t_fieldcat_alv WITH HEADER LINE.
DATA: t_fieldcat_mapa TYPE slis_t_fieldcat_alv WITH HEADER LINE.
DATA: v_layout_mapa   TYPE slis_layout_alv.
*--------------------------------------------------------------------*
* Variáveis para variante e inicialização
*--------------------------------------------------------------------*
DATA: gs_variante    TYPE disvariant,
      gs_defvariante TYPE disvariant.
**----------------------------------------------------------------------*
** Parâmetros gerais para as funções do ALV
**----------------------------------------------------------------------*
** Parâmetros gerais do layout do relatório
*DATA: gs_layout_i   TYPE slis_layout_alv.                " Layout de saída
**DATA: gt_sort_i     TYPE slis_sortinfo_alv OCCURS 0 WITH HEADER LINE.
** Auxiliares na montagem do lay-out do relatório na funçao do "ALV".
**DATA: gs_header_i   TYPE kkblo_listheader,
**      gt_header_i   TYPE kkblo_t_listheader.
*
** Para o catálogo de campos
*DATA: gt_fieldcat_i TYPE slis_t_fieldcat_alv WITH HEADER LINE.

************************************************************************
**  Definição de Tabelas Internas                                     **
************************************************************************
DATA: t_carac    LIKE zrnmme_caractov OCCURS 0 WITH HEADER LINE.

* tabela de previsão de produção
DATA: BEGIN OF t_zrnppt_prev OCCURS 0,
        werks          LIKE  zrnppt_prev-werks,
        bismt          LIKE  zrnppt_prev-bismt,
        ano            LIKE  zrnppt_prev-ano,
        mes            LIKE  zrnppt_prev-mes,
        matnr          LIKE  zrnppt_prev-matnr,
        espes          LIKE  zrnppt_prev-espes,
        larg           LIKE  zrnppt_prev-larg,
        compr          LIKE  zrnppt_prev-compr,
        base_maior     LIKE  zrnppt_prev-base_maior,
        base_menor     LIKE  zrnppt_prev-base_menor,
        codigo         LIKE  zrnppt_prev-codigo,
        prod           LIKE  zrnppt_prev-prod,
        pn             LIKE  zrnppt_prev-pn,
* Bruno Quadros - CSC - 30.09.2010 - Inicio
        entr           LIKE  zrnppt_prev-entr,
        peso_peca      LIKE  zrnppt_prev-peso_peca,
* Bruno Quadros - CSC - 30.09.2010 - Fim
        embalagem      LIKE  zrnppt_prev-embalagem,
        op1            LIKE  zrnppt_prev-op1,
        op2            LIKE  zrnppt_prev-op2,
        op3            LIKE  zrnppt_prev-op3,
        op4            LIKE  zrnppt_prev-op4,
        op5            LIKE  zrnppt_prev-op5,
        op6            LIKE  zrnppt_prev-op6,
        op7            LIKE  zrnppt_prev-op7,
        op8            LIKE  zrnppt_prev-op8,
        peso           LIKE  zrnppt_prev-peso,
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
        data_modif     LIKE zrnppt_prev-data_modif,
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim
        ferramenta     LIKE  zrnppt_prev-ferramenta,
        kunnr          LIKE  zrnppt_prev-kunnr,
        tamanho_lote   LIKE zrnppt_prev-tamanho_lote,
        estoque_minimo LIKE zrnppt_prev-estoque_minimo,
        estoque_min_pc LIKE zrnppt_prev-estoque_min_pc,
* Bruno Quadros - CSC - 12.04.2011 - Inicio
        peca           LIKE zrnppt_prev-peca,
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
        ativo          LIKE zrnppt_prev-ativo,
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
* Bruno Quadros - CSC - 12.04.2011 - Fim
        status_pcp     TYPE zrnppt_prev-status_pcp,
        status_om      TYPE zrnppt_prev-status_om,
      END OF t_zrnppt_prev.

DATA: t_zrnppt_prev_aux  LIKE TABLE OF t_zrnppt_prev WITH HEADER LINE,
      t_zrnppt_prev_aux2 LIKE TABLE OF t_zrnppt_prev WITH HEADER LINE,
      t_mslb             TYPE TABLE OF mslb,
      t_mslb_ltan        TYPE TABLE OF mslb,
      t_mslb_aux         TYPE TABLE OF mslb.
***Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Início
*DATA: r_ptnumber  TYPE RANGE OF zsuppt_mslb-partnumber,
*      wa_ptnumber LIKE LINE OF r_ptnumber.
***Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Fim


** tabela de saída que será impressa
*DATA: BEGIN OF t_saida OCCURS 0,
*       cliente    TYPE char10,
*       descricao  TYPE char40,
*       ferramenta LIKE zrnppt_prev-ferramenta,
*       roteiro    TYPE char30,
*       embalagem  LIKE zrnppt_prev-embalagem,
*       prev_org   LIKE zrnppt_prev-peso,
*       aproduzir  LIKE zrnppt_prev-peso,
*       pendencia  LIKE zrnppt_prev-peso,
*       ov         LIKE zrnppt_prev-peso,
*       op         LIKE zrnppt_prev-peso,
*       op_aber    LIKE zrnppt_prev-peso,
*       op_lib     LIKE zrnppt_prev-peso,
*       expedicao  TYPE zrnppt_prev-peso,
*       faturado   TYPE zrnppt_prev-peso,
*       faturado_cen TYPE zrnppt_prev-peso,   "Faturado em Outro Centro
*       faturado1  TYPE zrnppt_prev-peso,
*       faturado1_cen TYPE zrnppt_prev-peso,   "Faturado em Outro Centro
*       prodtot    TYPE zrnppt_prev-peso,
*       matnr      TYPE zrnppt_prev-matnr,
*       codigo     TYPE zrnppt_prev-codigo,
*       projeto    TYPE zsdt_part_number-projeto,
*       cod_cli    LIKE kna1-kunnr,
*       strgr      LIKE marc-strgr,
*       tamanho_lote LIKE zrnppt_prev-tamanho_lote,
*       estoque_minimo LIKE zrnppt_prev-estoque_minimo,
*       analise_prev   LIKE zrnppt_prev-estoque_minimo,
*       necessidade_mp LIKE zrnppt_prev-estoque_minimo,
*       saldo_mp      TYPE zrnppt_prev-peso,
*       mat_cq        TYPE zrnppt_prev-peso,
*       analise       LIKE zrnppt_prev-estoque_minimo,
*       tempo_plan    TYPE p DECIMALS 2,
*       materia_mp    TYPE zrnppt_prev-matnr,
** Bruno Quadros - CSC - 30.09.2010 - Inicio
*       abast         TYPE p,
*       abast_pec     TYPE p,
**       peso_peca     LIKE zrnppt_prev-peso_peca,
*       peso_peca     TYPE p DECIMALS 3,
*       exp_inter     TYPE zrnppt_prev-peso,
*       est_min_pec   TYPE p,
*       prev_org_pec  TYPE p,
*       ov_emitida    LIKE vbep-wmeng,
*       ov_emitida_pc TYPE p,
*       ov_peca       TYPE p,
*       faturado_pc   TYPE p,
*       faturado1_pc  TYPE p,
*       expedi_cen    TYPE zrnppt_prev-peso,
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
*       data_modif    TYPE zrnppt_prev-data_modif,
*       werks         TYPE crhd-werks,
*       meins         TYPE mara-meins,
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim
*       exp_inter_pec TYPE p,
*       expedicao_pec TYPE p,
*       mat_cq_pec    TYPE p,
*       prodtot_pec   TYPE p,
*       aproduzir_pec TYPE p,
*       pendencia_pec TYPE p,
*       op_lib_pec    TYPE p,
*       op_aber_pec   TYPE p,
*       analise_prev_pec TYPE p,
*       qtd_rack      TYPE p,
*       wrk02         LIKE t460a-wrk02,       "centro produtor
*       materia_prima LIKE mara-matnr,
****Inserção - Hermiro Carvalho - RDM 140238 - 22/06/2015 - Início
*       estoque_mp    LIKE mchb-clabs,
*       estoque_mp_cen    LIKE mchb-clabs,
*       estoque_ltan  LIKE mchb-clabs,
*       estoque_ltal  LIKE mchb-clabs,
****Inserção - Hermiro Carvalho - RDM 140238 - 22/06/2015 - Fim
*       max_expe_ton TYPE p DECIMALS 3,
*       max_expe_pc  TYPE p,
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
*       nivel_est(13)    TYPE c,
*       h_material       TYPE zrnppt_prev-prod,
*       pn               TYPE zrnppt_prev-pn,
*       l_entrega        TYPE kna1-sortl,
****Inserção - Hermiro Carvalho - RDM - 10/04/2015 - Início
*       periodo_1        TYPE zrnppt_prev-peso,
*       periodo_2        TYPE zrnppt_prev-peso,
****Inserção - Sidney Q de Andrade - RDM 164690 - 27/06/2016 - Inicio
*       periodo_3        TYPE zrnppt_prev-peso,
*       periodo_4        TYPE zrnppt_prev-peso,
*       periodo_5        TYPE zrnppt_prev-peso,
****Inserção - Sidney Q de Andrade - RDM 164690 - 27/06/2016 - Fim
*       periodo_1pc      TYPE p,
*       periodo_2pc      TYPE p,
****Inserção - Sidney Q de Andrade - RDM 164690 - 27/06/2016 - Inicio
*       periodo_3pc      TYPE p,
*       periodo_4pc      TYPE p,
*       periodo_5pc      TYPE p,
****Inserção - Sidney Q de Andrade - RDM 164690 - 27/06/2016 - Fim
****Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Início
*       ativo            TYPE zrnppt_prev-ativo,
*       planejadas       TYPE plaf-gsmng,
****Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Início
*       planejadas_pc    TYPE p,
****Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Fim
*       data_inicio      TYPE plaf-psttr,
*       produzir_op      TYPE afko-gamng,
*       prazo            TYPE caufv-gstrs,
****Inserção - Hermiro Carvalho - RDM - 10/04/2015 - Fim
****Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
*       est_min_dias     TYPE zsuppt_estoq_dia-stock_min,
**       est_min_pc       TYPE zrnppt_prev-estoque_min_pc,
****Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
*       cellcolor        TYPE lvc_t_scol,    "Estrutura para colorir células
****Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Início
*       ov_md81(17)      TYPE c,
*       pn_interno       TYPE zqmt_partnumber-partnumber,
*       datefrom         TYPE zqmt_partnumber-datefrom,
*       dateto           TYPE zqmt_partnumber-dateto,
****Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Fim
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim
*       mark(1),
*       faturado_cen_pc   TYPE p,
*       faturado1_cen_pc  TYPE p,
*       expedi_cen_pc     TYPE p,
** Bruno Quadros - CSC - 12.04.2011 - Fim
*       requisicoes       TYPE eban-menge,
*       requisicoes_pc    TYPE p,
*       pedidos           TYPE ekpo-menge,
*       fornecedor        TYPE ekko-lifnr,
*       pedidos_pc        TYPE p,
*       kdmat             TYPE vbap-kdmat,
*       producao(9)       TYPE c,
*       saldo_pedido      TYPE p DECIMALS 3,
****Inserção - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Início
*       estoque_forn      TYPE mslb-lblab,
**       expedicao_forn    TYPE zsuppt_mslb-expedicao_forn,
**       mp_forn           TYPE zsuppt_mslb-expedicao_forn,
**       mp_fis_forn       TYPE zsuppt_mslb-mp_forn,
**       processo_forn     TYPE zsuppt_mslb-processo_forn,
*       expedicao_forn     TYPE p DECIMALS 3,
*       mp_forn            TYPE p DECIMALS 3,
*       mp_fis_forn        TYPE p DECIMALS 3,
*       processo_forn      TYPE p DECIMALS 3,
****Inserção - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Fim
*       estfat            TYPE p DECIMALS 3,
*       vol_aproduzir      TYPE p DECIMALS 3,
*       vol_aprogramar    TYPE p DECIMALS 3,
*       estoque1dia       TYPE p DECIMALS 3,
*       ceex(19)          TYPE c,
*       cfat(19)          TYPE c,
*       cpdc(19)          TYPE c,
*       creq(19)          TYPE c,
*       ctot(19)          TYPE c,
*       scee(19)          TYPE c,
****Inserção - Hermiro Carvalho - RDM 172372 - 30/11/2016 - Início
*       desc_mp           TYPE char40,
****Inserção - Hermiro Carvalho - RDM 172372 - 30/11/2016 - Fim
****Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Início
*       estoque_forn_ltan  TYPE mslb-lblab,
****Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Fim
*       op_aber_cen  LIKE zrnppt_prev-peso,"Op aberta em outro centro
*       op_aber_cen_pec LIKE zrnppt_prev-peso,
*       op_lib_cen LIKE zrnppt_prev-peso,
*       op_lib_cen_pec LIKE zrnppt_prev-peso,
*       observacao     TYPE char30,
*       call_obs       TYPE char05,
**RDM 203086
*       lgort          TYPE marc-lgpro,
**RDM 203086
*       cobfat_exp_org(20)          TYPE c,
*       cob_exp_org(20)             TYPE c,
*       cob_fat_org(20)             TYPE c,
*       cob_op_total_dem_org(20)    TYPE c,
*       cob_mp_forne_dem_org(20)    TYPE c,
*       cob_ped_comp_org(20)        TYPE c,
*       cob_exp_60(20)              TYPE c,"Cobertura Expedição 60 dias
*       cob_stat_exp_60(20)         TYPE c,"Status Cobertura Expedicao 60 dias
*       statu_org(20)               TYPE c,"status Org
*       qtde_sku                    TYPE i,"Quantidade de SKU
*       porc_org(4)                 TYPE c,"Porcentagem Org.
*       s_mp                        TYPE i,"S/MP.
*       status_60(20)               TYPE c,"Status SKU
*       qtde_sku_60                 TYPE i,"Qtde. SKU 60 dias.
*       porc_org_60(4)              TYPE c,"% 60 dias.
*       cobfat_exp_60(20)           TYPE c,"Cobertura Fat+Expedicao 60 dias.
*       status_cobert_exp_org(20)   TYPE c,"STATUS COBERTURA Expedição ORG
*       vol_mensal_atendido(20)     TYPE c,"VOLUME MENSAL ATENDIDO
*       soma_mps(8)                 TYPE c,
*END OF t_saida.
*
*DATA: gt_saida LIKE TABLE OF t_saida.
*
*DATA: t_saida_aux LIKE STANDARD TABLE OF t_saida WITH HEADER LINE.

***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Início
TYPES: BEGIN OF ty_plaf_aux,
         plnum TYPE plaf-plnum,
         matnr TYPE plaf-matnr,
         plwrk TYPE plaf-plwrk,
         gsmng TYPE plaf-gsmng,
         psttr TYPE plaf-psttr,
         meins TYPE plaf-meins,
         kdauf TYPE plaf-kdauf,
         kdpos TYPE plaf-kdpos,
       END OF ty_plaf_aux.
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Fim

***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
TYPES: BEGIN OF ty_zsuppt_estoq_dia,
         matnr     TYPE zsuppt_estoq_dia-matnr,
         werks     TYPE zsuppt_estoq_dia-werks,
         kunnr     TYPE zsuppt_estoq_dia-kunnr,
         stock_min TYPE zsuppt_estoq_dia-stock_min,
       END OF ty_zsuppt_estoq_dia.
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim

***Inserção - LG - RDM 334177 - 15/04/2026 - Início - Campos EF
TYPES: BEGIN OF ty_calc_campos_ef,
         zzdemadtot   TYPE p DECIMALS 3,     "Demanda Atual Total (ORG+TN1+TN2)
         zzprodtot    TYPE p DECIMALS 3,     "Em Produção Total
         zzexpdtot    TYPE p DECIMALS 3,     "Expedição Total
         zzfattot     TYPE p DECIMALS 3,     "Faturado Total
         zzprodexpfat TYPE p DECIMALS 3,     "Produção + Expedição + Faturado
         zzdelta      TYPE p DECIMALS 3,     "Delta
         zzopsemmov   TYPE i,                "Total OP Sem Movimentação
         zzpedsemmov  TYPE i,                "Total Pedido Sem Movimentação
         zzacao(200)  TYPE c,                "Ação
         zzfatorg3    TYPE p DECIMALS 3,     "Faturado Org -3 (M-3)
         zzfatorg2    TYPE p DECIMALS 3,     "Faturado Org -2 (M-2)
         zzfatorg1    TYPE p DECIMALS 3,     "Faturado Org -1 (M-1)
         zzfattothist TYPE p DECIMALS 3,     "Faturado Total (Histórico)
       END OF ty_calc_campos_ef.

TYPES: BEGIN OF ty_ef_fat_hist_sum,
         matnr TYPE mara-matnr,
         werks TYPE crhd-werks,
         kunag TYPE vbrk-kunag,
         m3    TYPE p DECIMALS 3,
         m2    TYPE p DECIMALS 3,
         m1    TYPE p DECIMALS 3,
       END OF ty_ef_fat_hist_sum.

TYPES: BEGIN OF ty_ef_sem_mov_sum,
         matnr TYPE mara-matnr,
         werks TYPE crhd-werks,
         qtd   TYPE i,
       END OF ty_ef_sem_mov_sum.

TYPES: BEGIN OF ty_ef_vbak_remessa,
         vbeln TYPE vbak-vbeln,
         vdatu TYPE vbak-vdatu,
       END OF ty_ef_vbak_remessa.

TYPES: BEGIN OF ty_ef_op_det,
         aufnr       TYPE aufk-aufnr,
         matnr       TYPE mara-matnr,
         qtd         TYPE afko-gamng,
         erdat       TYPE aufk-erdat,
         objnr       TYPE aufk-objnr,
         diario      TYPE zrnppt_dpo-numdiar,
         werks       TYPE caufv-werks,
         planejado   TYPE caufv-gamng,
         produzido   TYPE zrnppt_pmvm-zqtreg,
         intermed    TYPE c LENGTH 1,
         confirmacao TYPE zrnppt_papo-zqtprd,
         status_op   TYPE c LENGTH 4,
         nfenum      TYPE j_1bnfdoc-nfenum,
       END OF ty_ef_op_det.

TYPES: BEGIN OF ty_ef_ped_det,
         ebeln  TYPE ekpo-ebeln,
         ebelp  TYPE ekpo-ebelp,
         matnr  TYPE mara-matnr,
         qtd    TYPE ekpo-menge,
         data   TYPE ekko-bedat,
         banfn  TYPE ekpo-banfn,
         werks  TYPE ekpo-werks,
         nfenum TYPE j_1bnfdoc-nfenum,
       END OF ty_ef_ped_det.

DATA: gt_ef_fat_hist_sum TYPE SORTED TABLE OF ty_ef_fat_hist_sum WITH UNIQUE KEY matnr werks kunag,
      gs_ef_fat_hist_sum TYPE ty_ef_fat_hist_sum,
      gt_ef_op_sem_mov   TYPE SORTED TABLE OF ty_ef_sem_mov_sum WITH UNIQUE KEY matnr werks,
      gs_ef_op_sem_mov   TYPE ty_ef_sem_mov_sum,
      gt_ef_ped_sem_mov  TYPE SORTED TABLE OF ty_ef_sem_mov_sum WITH UNIQUE KEY matnr werks,
      gs_ef_ped_sem_mov  TYPE ty_ef_sem_mov_sum,
      gt_ef_vbak_remessa TYPE SORTED TABLE OF ty_ef_vbak_remessa WITH UNIQUE KEY vbeln,
      gs_ef_vbak_remessa TYPE ty_ef_vbak_remessa,
      gt_ef_op_det       TYPE TABLE OF ty_ef_op_det,
      gs_ef_op_det       TYPE ty_ef_op_det,
      gt_ef_ped_det      TYPE TABLE OF ty_ef_ped_det,
      gs_ef_ped_det      TYPE ty_ef_ped_det,
      gv_ef_precalc      TYPE c.
***Inserção - LG - RDM 334177 - 15/04/2026 - Fim

DATA: BEGIN OF t_saida_i OCCURS 0,
        werks           LIKE crhd-werks,
        matnr_charg     LIKE zrnppt_pmvm-zmatnr,
        charg           LIKE zrnppt_pmvm-zcharg,
        znumoa          LIKE zrnppt_pmvm-znumoa,
        znumpp          LIKE zrnppt_pmvm-znumpp,
        ov_it(18)       TYPE c,
        faturamento(18) TYPE c,
        peso            LIKE zrnppt_prev-peso,
        peca            TYPE p,
        nf              LIKE vbrk-xblnr,
* Bruno Quadros - CSC - 29.07.2011 - Inicio
        dt_prod         LIKE sy-datum,
        dt_fatu         LIKE sy-datum,
        matnr           LIKE mara-matnr,
        materia_prima   LIKE mara-matnr,
        requisicao      LIKE eban-banfn,
        peso_ret        LIKE eban-menge,
        pedido          LIKE ekko-ebeln,
***Inserção - Hermiro Carvalho - RDM 158724 - 29/02/2016 - Início
        saldo_pedido    LIKE eban-menge,
        num_pl_corte    TYPE zrnppt_pl_corte-num_pl_corte,
        sortl           TYPE lfa1-sortl,
        roteiro         TYPE char30,
        dt_zpetiq       TYPE sy-datum,
        wrk02           LIKE crhd-werks,  " RDM 177618
***Inserção - Hermiro Carvalho - RDM 158724 - 29/02/2016 - Fim
* >>> - 08.10.2019 17:21:06 - RDM 225838 - Simone
        estqtran(9)     TYPE c,
* <<< - 08.10.2019 17:21:06 - RDM 225838 - Simone
* Bruno Quadros - CSC - 29.07.2011 - Fim
        status_pcp      TYPE zrnppt_prev-status_pcp,
        status_om       TYPE zrnppt_prev-status_om,
*RDM322490 - GVCA20241122 - Begin
        erdat           TYPE aufk-erdat,
        ernam           TYPE aufk-ernam,
*RDM322490 - GVCA20241122 - End
      END OF t_saida_i.

DATA: gt_saida_i LIKE TABLE OF t_saida_i.

***Inserção - Hermiro Carvalho - RDM 105472 - 27/08/2014 - Início
*TYPES: BEGIN OF ty_stpo_aux,
*         stlnr   TYPE stpo-stlnr,
*         idnrk   TYPE stpo-idnrk,
*         meins   TYPE stpo-meins,
*         menge   TYPE stpo-menge,
*       END OF ty_stpo_aux.
*DATA:  t_stpo_aux  TYPE TABLE OF ty_stpo_aux.
*DATA:  wa_stpo_aux TYPE ty_stpo_aux.
*DATA:  t_stpo_log  TYPE TABLE OF ty_stpo_aux.
*DATA:  wa_stpo_log TYPE ty_stpo_aux.
***Inserção - Hermiro Carvalho - RDM 105472 - 27/08/2014 - Fim

***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
TYPES: BEGIN OF ty_kna1,
         kunnr TYPE kna1-kunnr,
         sortl TYPE kna1-sortl,
       END OF ty_kna1.
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim

* Documento de faturamento: dados de cabeçalho
DATA: BEGIN OF t_vbrk OCCURS 0,
        vbeln LIKE vbrk-vbeln,
        erdat LIKE vbrk-erdat,
        kunag LIKE vbrk-kunag,
        fksto LIKE vbrk-fksto,
        ntgew LIKE vbrp-ntgew,
        matnr LIKE vbrp-matnr,
      END OF t_vbrk.

* Documento de faturamento: dados de item
DATA: BEGIN OF t_vbrp OCCURS 0,
        vbeln      LIKE vbrp-vbeln,
        posnr      LIKE vbrp-posnr,
        aubel      LIKE vbrp-aubel,
        aupos      LIKE vbrp-aupos,
        matnr      LIKE vbrp-matnr,
        ntgew      LIKE vbrp-ntgew,
* Bruno Quadros - CSC - 12.04.2011 - Inicio
        fkimg      LIKE vbrp-fkimg,
        werks      LIKE vbrp-werks,
        erdat      LIKE vbrp-erdat,
        xblnr      LIKE vbrk-xblnr,
* Bruno Quadros - CSC - 12.04.2011 - Fim
        kunag      LIKE vbrk-kunag,
        kdmat      LIKE vbap-kdmat,
        ferramenta LIKE zrnppt_prev-ferramenta,
        matcli(30),
      END OF t_vbrp.


DATA: BEGIN OF it_vbrp OCCURS 0,
        vbeln      LIKE vbrp-vbeln,
        posnr      LIKE vbrp-posnr,
        vgbel      LIKE vbrp-vgbel,
        aubel      LIKE vbrp-aubel,
        aupos      LIKE vbrp-aupos,
        matnr      LIKE vbrp-matnr,
        ntgew      LIKE vbrp-ntgew,
        fkimg      LIKE vbrp-fkimg,
        werks      LIKE vbrp-werks,
        erdat      LIKE vbrp-erdat,
        xblnr      LIKE vbrk-xblnr,
        kunag      LIKE vbrk-kunag,
        kdmat      LIKE vbap-kdmat,
        ferramenta LIKE zrnppt_prev-ferramenta,
        matcli(30),
      END OF it_vbrp.

* Atribuição de planos a materiais
TYPES: BEGIN OF e_mapl,
         matnr TYPE mapl-matnr,
         werks TYPE mapl-werks,
         plnty TYPE mapl-plnty,
         plnnr TYPE mapl-plnnr,
         plnal TYPE mapl-plnal,
         zkriz TYPE mapl-zkriz,
         zaehl TYPE mapl-zaehl,
       END OF e_mapl.

* Plano: operação
TYPES: BEGIN OF e_plpo,
         plnnr TYPE plpo-plnnr,
         plnkn TYPE plpo-plnkn,
         vornr TYPE plpo-vornr,
         arbid TYPE plpo-arbid,
         ltxa1 TYPE plpo-ltxa1,
       END OF e_plpo.

* Cabeçalho do centro de trabalho
TYPES: BEGIN OF e_crhd,
         objid TYPE crhd-objid,
         arbpl TYPE crhd-arbpl,
         ktsch TYPE crhd-ktsch,
       END OF e_crhd.

* Roteiro - seleção de itens
TYPES: BEGIN OF e_plas,
         plnnr TYPE plas-plnnr,
         plnal TYPE plas-plnal,
         plnkn TYPE plas-plnkn,
         zaehl TYPE plas-zaehl,
       END OF e_plas.

DATA: t_plas TYPE TABLE OF e_plas WITH HEADER LINE.

DATA: t_mapl TYPE TABLE OF e_mapl WITH HEADER LINE.

DATA: t_plpo TYPE TABLE OF e_plpo WITH HEADER LINE.

DATA: t_plpo_aux TYPE TABLE OF e_plpo WITH HEADER LINE.

DATA: t_crhd_aux TYPE TABLE OF e_crhd WITH HEADER LINE.

***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
DATA: t_bdc        TYPE TABLE OF bdcdata,
      t_return_bdc TYPE TABLE OF bdcmsgcoll.

***Inserção - LG - RDM 334177 - 15/04/2026 - Início - Estruturas de Suporte EF
TYPES: BEGIN OF ty_demanda_ef,
         matnr         TYPE mara-matnr,
         werks         TYPE crhd-werks,
         prev_org_atua TYPE p DECIMALS 3,     "Previsão ORG Atual
         prev_tn1_atua TYPE p DECIMALS 3,     "Previsão TN+1 Atual
         prev_tn2_atua TYPE p DECIMALS 3,     "Previsão TN+2 Atual
       END OF ty_demanda_ef.

TYPES: BEGIN OF ty_producao_exp_fat,
         matnr            TYPE mara-matnr,
         werks            TYPE crhd-werks,
         ped_compra_ton   TYPE p DECIMALS 3,     "Pedidos Compra TON
         ped_compra_outro TYPE p DECIMALS 3,     "Pedidos Compra Outro Centro TON
         op_aber          TYPE p DECIMALS 3,     "OP Aberta
         op_aber_outro    TYPE p DECIMALS 3,     "OP Aberta Outro Centro
         expedicao_ton    TYPE p DECIMALS 3,     "Expedição
         expedicao_outro  TYPE p DECIMALS 3,     "Expedição Outro Centro
         faturado_ton     TYPE p DECIMALS 3,     "Faturado
         faturado_outro   TYPE p DECIMALS 3,     "Faturado Outro Centro
       END OF ty_producao_exp_fat.

TYPES: BEGIN OF ty_faturado_historico,
         matnr    TYPE mara-matnr,
         werks    TYPE crhd-werks,
         data_fat TYPE vbrk-fkdat,          "Data de Faturamento
         data_rem TYPE vbak-vdatu,          "Data de Remessa
         ton      TYPE p DECIMALS 3,        "Tonelagem
         mes_ref  TYPE c LENGTH 7,          "Mês de Referência (MM/YYYY)
       END OF ty_faturado_historico.

TYPES: BEGIN OF ty_op_sem_mov,
         werks    TYPE crhd-werks,
         matnr    TYPE mara-matnr,
         znumpp   TYPE zrnppt_popa-znumpp,  "Ordem PP
         objnr    TYPE aufk-objnr,         "OBJNR da OP
         aufnr    TYPE aufk-aufnr,         "Número OP
         status   TYPE c LENGTH 4,          "Status (ABER/LIB)
         tem_forn TYPE c LENGTH 1,          "Tem status FORN (X=sim)
         tem_conf TYPE c LENGTH 1,          "Tem confirmação (X=sim)
       END OF ty_op_sem_mov.

TYPES: BEGIN OF ty_ped_sem_mov,
         werks    TYPE crhd-werks,
         matnr    TYPE mara-matnr,
         ebeln    TYPE ekko-ebeln,         "Número Pedido
         ebelp    TYPE ekpo-ebelp,         "Item Pedido
         menge    TYPE ekpo-menge,         "Quantidade
         tem_ekbe TYPE c LENGTH 1,         "Existe em EKBE (X=sim)
       END OF ty_ped_sem_mov.

DATA: t_demanda_ef          TYPE TABLE OF ty_demanda_ef,
      wa_demanda_ef         TYPE ty_demanda_ef,
      t_producao_exp_fat    TYPE TABLE OF ty_producao_exp_fat,
      wa_producao_exp_fat   TYPE ty_producao_exp_fat,
      t_faturado_historico  TYPE TABLE OF ty_faturado_historico,
      wa_faturado_historico TYPE ty_faturado_historico,
      t_op_sem_mov          TYPE TABLE OF ty_op_sem_mov,
      wa_op_sem_mov         TYPE ty_op_sem_mov,
      t_ped_sem_mov         TYPE TABLE OF ty_ped_sem_mov,
      wa_ped_sem_mov        TYPE ty_ped_sem_mov,
      t_calc_campos_ef      TYPE TABLE OF ty_calc_campos_ef,
      wa_calc_campos_ef     TYPE ty_calc_campos_ef.
***Inserção - LG - RDM 334177 - 15/04/2026 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
DATA: t_zsuppt_estoq_dia TYPE TABLE OF ty_zsuppt_estoq_dia.
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Início
DATA: t_plaf_aux  TYPE TABLE OF ty_plaf_aux,
      wa_plaf_aux TYPE ty_plaf_aux.
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim

* Plano - meios auxiliares de produção
DATA: BEGIN OF t_plfh OCCURS 0,
        plnty LIKE plfh-plnty,
        plnnr LIKE plfh-plnnr,
        objty LIKE plfh-objty,
        objid LIKE plfh-objid,
        plnkn LIKE plfh-plnkn,
      END OF t_plfh.

* Dados de centro para material
DATA: BEGIN OF t_marc OCCURS 0,
        matnr LIKE marc-matnr,
        werks LIKE marc-werks,
        strgr LIKE marc-strgr,
        minbe LIKE marc-minbe,
* Bruno Quadros - CSC - 29.07.2011 - Inicio
        bstma LIKE marc-bstma,
* Bruno Quadros - CSC - 29.07.2011 - Fim
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Início
        shzet LIKE marc-shzet,
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Fim
*RDM 203086
        lgpro LIKE marc-lgpro,
        sobsl LIKE marc-sobsl,
*RDM 203086
      END OF t_marc.

* Estoque de ordens-cliente
DATA: BEGIN OF t_mska OCCURS 0,
        werks      LIKE mska-werks,
        lgort      LIKE mska-lgort,
        matnr      LIKE mska-matnr,
        charg      LIKE mska-charg,
        kalab      LIKE mska-kalab,
        ersda      LIKE mska-ersda,
        kains      LIKE mska-kains,
        vbeln      LIKE mska-vbeln,
        posnr      LIKE mska-posnr,
        kdmat      LIKE vbap-kdmat,
        portao(5)  TYPE c,
        ov_separa  TYPE char1,    "  'X' se foi lido da OV Separa
        sortl      LIKE kna1-sortl,
        ferramenta LIKE zrnppt_prev-ferramenta,
        matcli(30),
        kunnr      LIKE vbak-kunnr,
      END OF t_mska.

* Ordem planejada
DATA: BEGIN OF t_plaf OCCURS 0,
        plnum  LIKE plaf-plnum,
        matnr  LIKE plaf-matnr,
        plwrk  LIKE plaf-plwrk,
        gsmng  LIKE plaf-gsmng,
        pedtr  LIKE plaf-pedtr,
        kdauf  LIKE plaf-kdauf,
        kdpos  LIKE plaf-kdpos,
        bdzei  LIKE mdpb-bdzei,
        bstdk  LIKE zrnppt_contrprog-bstdk,
        meins  LIKE plaf-meins,
        erdat  LIKE vbak-erdat,
        kwmeng LIKE vbap-kwmeng,
        cuobj  LIKE vbap-cuobj,
        untto  LIKE vbap-untto,
        arktx  LIKE vbap-arktx,
        cmgst  LIKE vbuk-cmgst,
        gbsta  LIKE vbup-gbsta,
        etenr  LIKE vbep-etenr,
        delkz  LIKE mdrq-delkz.
DATA: END OF t_plaf.

* Dados mestre da ordem
DATA: BEGIN OF t_aufk OCCURS 0,
        aufnr    LIKE aufk-aufnr,
        auart    LIKE aufk-auart,
        kdauf    LIKE aufk-kdauf,
        kdpos    LIKE aufk-kdpos,
        objnr    LIKE aufk-objnr,
        gltrp    LIKE afko-gltrp,
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Início
        gstrs    LIKE afko-gstrs,
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Fim
        aufpl    LIKE afko-aufpl,
        gamng    LIKE afko-gamng,
        gmein    LIKE afko-gmein,
        matnr    LIKE afpo-matnr,
        cy_seqnr LIKE caufv-cy_seqnr,
* Bruno Quadros - CSC - 13.04.2011 - Inicio
        werks    LIKE vbap-werks,
        znumoa   LIKE zrnppt_pmvm-znumoa,
* Bruno Quadros - CSC - 13.04.2011 - Fim
        qte_ov   LIKE vbap-kwmeng,   "Qtde.Ord.Cli
        untto    LIKE vbap-untto,
        cod_cli  LIKE kna1-kunnr.
DATA: END OF t_aufk.

DATA: wa_plaf LIKE t_plaf.

***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
DATA: wa_kna1 TYPE ty_kna1,
      wa_bdc  TYPE bdcdata.
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
DATA: wa_zsuppt_estoq_dia TYPE ty_zsuppt_estoq_dia.
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim

DATA: t_aufk_aux  LIKE TABLE OF t_aufk WITH HEADER LINE.

DATA: BEGIN OF t_operacao OCCURS 0,
        steus LIKE afvc-steus,
        arbpl LIKE crhd-arbpl,
      END OF t_operacao.

DATA: BEGIN OF t_vbak OCCURS 0,
        vbeln LIKE vbak-vbeln,
        kunnr LIKE vbak-kunnr,
        auart LIKE vbak-auart,
      END OF t_vbak.

DATA: BEGIN OF t_kna1 OCCURS 0,
        kunnr LIKE kna1-kunnr,
        sortl LIKE kna1-sortl,
      END OF t_kna1.

* Tabela interna que contem os dados do Relatório
DATA: BEGIN OF t_dad_rel OCCURS 0,
        cn_tra     LIKE   crhd-arbpl,  "Centro de Trabalho
        sortl      LIKE   kna1-sortl,  "Cliente
        dat_pz     LIKE   sy-datum,    "Prazo de Entrega
        nr_ov      LIKE   vbak-vbeln,  "Ordem de Venda
        it_ov      LIKE   vbap-posnr,  "Item da Ordem de Vd.
        bdzei      LIKE   mdpb-bdzei,  "Gestão de Demanda.
        matnr      LIKE   mdpb-matnr,  "Material.
        delkz      LIKE   mdrq-delkz,
        qte_ov     LIKE   vbap-kwmeng, "Qtde.Ord.Cli
        kdmat      LIKE   vbap-kdmat,
        or_pla     LIKE   plaf-plnum,  "Ordem Planejada
        or_pro     LIKE   aufk-aufnr,  "Ordem de Produção
        qte_pl     LIKE   plaf-gsmng,  "Quantidade da Ordem
        qte_or     LIKE   plaf-gsmng,  "Quantidade da Ordem
        tipo(1),                       "Quantidade da Ordem
        ferramenta LIKE   crvm_b-matnr, "Num. da Ferramenta
        kunnr      LIKE   kna1-kunnr,
        codigo(30),
        op_aber    LIKE zrnppt_prev-peso,
        op_lib     LIKE zrnppt_prev-peso,
        op         LIKE zrnppt_pmvm-znumoa,
        op_peca    TYPE p,
        werks      TYPE plaf-plwrk,
        werks2     TYPE plaf-plwrk, " RDM 177134
      END OF t_dad_rel.

** Tabela interna de centro de trabalho com a Chave de Controle igual
** a PP01
DATA: BEGIN OF t_cent_trab OCCURS 0,
        objty      LIKE  crhd-objty,      "Ctg.de objeto
        objid      LIKE  crhd-objid,      "ID do objeto
        arbpl      LIKE  crhd-arbpl,      "Centro de Trabalho
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
        werks      LIKE  crhd-werks,      "Centro
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
        steus      LIKE  crhd-steus,      "Tipo de C. trabalho
        ktext      LIKE  crtx-ktext,      "Descrição do centro
        ferramenta LIKE crvm_b-matnr,  "Ferramenta da PB
      END OF t_cent_trab.

* Tabela Interna que armazena os Números de Materiais do tipo
* FERT, ZALT, ZALP, ZALE, ZALC, ZACT e ZAPT
DATA: BEGIN OF t_tip_mat OCCURS 0,
        matnr LIKE mara-matnr,
        mtart LIKE mara-mtart,
      END OF t_tip_mat.

***Inserção - Hermiro Carvalho - RDM 162929 - 23/05/2016 - Início
DATA: lc_return TYPE c,
      lt_fields TYPE TABLE OF sval,
      ls_fields TYPE sval.
***Inserção - Hermiro Carvalho - RDM 162929 - 23/05/2016 - Fim
* Tabela Interna para extrair as Datas de Prazo de Entrega
DATA: BEGIN OF t_imdrqx OCCURS 0.
        INCLUDE STRUCTURE mdrq.
      DATA: END OF t_imdrqx.

DATA: BEGIN OF wa_imdrqx.
        INCLUDE STRUCTURE mdrq.
      DATA: END OF wa_imdrqx.

DATA: BEGIN OF t_emdpsx OCCURS 0.
        INCLUDE STRUCTURE mdps.
      DATA: END OF t_emdpsx.

* Estoques de lotes
DATA: BEGIN OF t_mchb OCCURS 0,
        werks      LIKE mchb-werks,
        lgort      LIKE mchb-lgort,
        matnr      LIKE mchb-matnr,
        charg      LIKE mchb-charg,
        clabs      LIKE mchb-clabs,
        ersda      LIKE mchb-ersda,
        cinsm      LIKE mchb-cinsm,
        cspem      LIKE mchb-cspem,
        bismt      LIKE mara-bismt,
        ov         LIKE zrnppt_pmvm-zordvenda,
        it         LIKE zrnppt_pmvm-zitem,
        znumpp     LIKE zrnppt_pmvm-znumpp,
        ferramenta LIKE zrnppt_prev-ferramenta,
        matcli(30),
        peca       LIKE zrnppt_pmvm-zqtpeca,
* Bruno Quadros - CSC - 29.07.2011 - Inicio
        zdatreg    LIKE zrnppt_pmvm-zdatreg,
* Bruno Quadros - CSC - 29.07.2011 - Fim
      END OF t_mchb.

TYPES: BEGIN OF ty_mchb1,
         werks      LIKE mchb-werks,
         lgort      LIKE mchb-lgort,
         matnr      LIKE mchb-matnr,
         charg      LIKE mchb-charg,
         clabs      LIKE mchb-clabs,
         ersda      LIKE mchb-ersda,
         cinsm      LIKE mchb-cinsm,
         cspem      LIKE mchb-cspem,
         bismt      LIKE mara-bismt,
         ov         LIKE zrnppt_pmvm-zordvenda,
         it         LIKE zrnppt_pmvm-zitem,
         znumpp     LIKE zrnppt_pmvm-znumpp,
         ferramenta LIKE zrnppt_prev-ferramenta,
         matcli(30),
         peca       LIKE zrnppt_pmvm-zqtpeca,
         zdatreg    LIKE zrnppt_pmvm-zdatreg,

       END OF ty_mchb1.

DATA:    wa_mchb TYPE ty_mchb1.

* Dados gerais de material
DATA: BEGIN OF t_mara OCCURS 0,
        matnr LIKE mara-matnr,
        bismt LIKE mara-bismt,
        meins LIKE mara-meins,
        spart LIKE mara-spart,
        kzkfg LIKE mara-kzkfg,
      END OF t_mara.

DATA: BEGIN OF t_marm OCCURS 0,
        matnr LIKE marm-matnr,
        meinh LIKE marm-meinh,
        umrez LIKE marm-umrez,
        umren LIKE marm-umren,
      END OF t_marm.

* Grazielle Alves - Procwork - Início - 12/09/2008 - Projeto Complementar - PP128
DATA: BEGIN OF t_crhd OCCURS 0,
        objid LIKE crhd-objid,
        arbpl LIKE crhd-arbpl,
      END   OF t_crhd.
* Grazielle Alves - Procwork - Fim    - 12/09/2008 - Projeto Complementar - PP128

RANGES:
        r_paart             FOR plaf-paart,
*PP - Cenário Estoque Consignado - RDM 239896
        r_werks_consignado  FOR mchb-werks.
*PP - Cenário Estoque Consignado - RDM 239896
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Inicio
*        r_cod_cli FOR kna1-kunnr,

*DATA: BEGIN OF t_cod_cli OCCURS 0,
*   kunnr LIKE kna1-kunnr,
*   sortl LIKE kna1-sortl,
*END OF t_cod_cli.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim

DATA: BEGIN OF t_pmvm OCCURS 0,
        znumoa      LIKE zrnppt_pmvm-znumoa,
        znumpp      LIKE zrnppt_pmvm-znumpp,
        zmatnr      LIKE zrnppt_pmvm-zmatnr,
        zcharg      LIKE zrnppt_pmvm-zcharg,
        zordvenda   LIKE zrnppt_pmvm-zordvenda,
        zitem       LIKE zrnppt_pmvm-zitem,
        zqtreg      LIKE zrnppt_pmvm-zqtreg,
        zqtreal     LIKE zrnppt_pmvm-zqtreal,
        zcodcliente LIKE zrnppt_pmvm-zcodcliente,
        zqtpeca     LIKE zrnppt_pmvm-zqtpeca,
        werks       LIKE zrnppt_pcoa-werks,
        zdatreg     LIKE zrnppt_pmvm-zdatreg,
        kdmat       LIKE vbap-kdmat,
        zmatcli(30),
      END OF t_pmvm.

* Bruno Quadros - CSC - 30.09.2010 - Inicio
DATA: BEGIN OF t_stpo OCCURS 0,
        stlty LIKE stpo-stlty,
        stlnr LIKE stpo-stlnr,
        stlkn LIKE stpo-stlkn,
        idnrk LIKE stpo-idnrk,
        meins LIKE stpo-meins,
        menge LIKE stpo-menge,
* Bruno Quadros - CSC - 29.07.2011 - Inicio
        matnr LIKE mara-matnr,
* Bruno Quadros - CSC - 29.07.2011 - Fim
      END OF t_stpo.
* Bruno Quadros - CSC - 30.09.2010 - Fim

* Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
DATA tl_phs_canc LIKE TABLE OF zrnppt_phs WITH HEADER LINE.  "Canceladas
* Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Fim

***Inserção - Hermiro Carvalho - RDM 119419 - 18/11/2014 - Início
TYPES: BEGIN OF ty_stpo,
         stlty TYPE stpo-stlty,
         stlnr TYPE stpo-stlnr,
         idnrk TYPE stpo-idnrk,
       END OF ty_stpo.

*       BEGIN OF ty_mast,
*         matnr    TYPE mast-matnr,
*         werks    TYPE mast-werks,
*         stlan    TYPE mast-stlan,
*         stlnr    TYPE mast-stlnr,
*         stlal    TYPE mast-stlal,
*       END OF ty_mast.

DATA: t_stpo2     TYPE TABLE OF ty_stpo,
      t_mast      TYPE TABLE OF ty_mast,
      "t_mast_log  TYPE TABLE OF mast,
      wa_stpo     TYPE ty_stpo,
      wa_mseg_log TYPE mseg,
      v_answer    TYPE c.
"   wa_mast     TYPE ty_mast,
"  wa_mast_log TYPE mast.
***Inserção - Hermiro Carvalho - RDM 119419 - 18/11/2014 - Fim

****Inserção - Hermiro Carvalho - RDM 140328 - 22/06/2015 - Início
TYPES: BEGIN OF ty_alternativa,
         werks     TYPE zsuppt_lista_alt-werks,
         matnr     TYPE zsuppt_lista_alt-matnr,
         charg     TYPE zsuppt_lista_alt-charg,
         matnr_pa  TYPE zsuppt_lista_alt-matnr_pa,
         obs       TYPE zsuppt_lista_alt-obs,
         clabs     TYPE zsuppt_lista_alt-clabs,
         dt_lib_qa TYPE zsuppt_lista_alt-dt_lib_qa,
         rsnum     TYPE zsuppt_lista_alt-rsnum,
       END OF ty_alternativa,

       BEGIN OF ty_hist_lt,
         werks TYPE zsuppt_hist_lt-werks,
         mantr TYPE zsuppt_hist_lt-matnr,
         idnrk TYPE zsuppt_hist_lt-idnrk,
       END OF ty_hist_lt,
*PP - Cenário Estoque Consignado - RDM 239896
       BEGIN OF ty_consignado,
         matnr TYPE mchb-matnr,
         werks TYPE mchb-werks,
         lgort TYPE mchb-lgort,
         charg TYPE mchb-charg,
         ersda TYPE mchb-ersda,
         clabs TYPE mchb-clabs,
       END OF ty_consignado,
*PP - Cenário Estoque Consignado - RDM 239896
       BEGIN OF ty_mchb,
         matnr TYPE mchb-matnr,
         werks TYPE mchb-werks,
         lgort TYPE mchb-lgort,
         charg TYPE mchb-charg,
         ersda TYPE mchb-ersda,
         clabs TYPE mchb-clabs,
         obs   TYPE zsuppt_lista_alt-obs,
       END OF ty_mchb,

       BEGIN OF ty_mchb_alv,
         matnr        TYPE mchb-matnr,
         werks        TYPE mchb-werks,
         lgort        TYPE mchb-lgort,
         charg        TYPE mchb-charg,
         ersda        TYPE mchb-ersda,
         clabs        TYPE mchb-clabs,
         obs          TYPE zsuppt_lista_alt-obs,
         aufnr        TYPE caufv-aufnr,
         znumoa_lib   TYPE zrnppt_pcoa-znumoa,
         znumoa_aber  TYPE zrnppt_pcoa-znumoa,
         cinsm        TYPE mchb-cinsm,
*PP - Melhorias MP Alternativa - RDM 271191
         resbp        TYPE zppt_pajt-resbp,
*PP - Melhorias MP Alternativa - RDM 271191
         lote_compart TYPE char18,
       END OF ty_mchb_alv.

TYPES: BEGIN OF ty_mseg,
         werks     TYPE mseg-werks,
         matnr     TYPE mseg-matnr,
         charg     TYPE mseg-charg,
         bwart     TYPE mseg-bwart,
         menge     TYPE mseg-menge,
         aufnr     TYPE mseg-aufnr,
         matnr_apl TYPE afpo-matnr,
       END OF ty_mseg.

DATA: t_alternativa  TYPE TABLE OF ty_alternativa,
      t_hist_lt      TYPE TABLE OF ty_hist_lt,
*PP - Cenário Estoque Consignado - RDM 239896
      t_consignado   TYPE TABLE OF ty_consignado,
*PP - Cenário Estoque Consignado - RDM 239896
      t_mchb_aux     LIKE TABLE OF t_mchb,
      t_mchb_aux2    LIKE TABLE OF t_mchb,
      t_mchb_alv     TYPE TABLE OF ty_mchb_alv,
      t_oest_log     TYPE TABLE OF zppt_oest_log,
      t_oest_log_aux TYPE TABLE OF zppt_oest_log,
      wa_alternativa TYPE ty_alternativa,
      wa_hist_lt     TYPE ty_hist_lt,
      wa_mchb_aux    LIKE t_mchb,
      wa_mchb_alv    TYPE ty_mchb_alv.

DATA: t_mseg      TYPE TABLE OF ty_mseg,
      wa_mseg     TYPE ty_mseg,
      t_mseg_sum  TYPE TABLE OF ty_mseg,
      wa_mseg_sum TYPE ty_mseg.
****Inserção - Hermiro Carvalho - RDM 140328 - 22/06/2015 - Fim

TYPES: BEGIN OF ty_vbak,
         vbeln TYPE vbak-vbeln,
         auart TYPE vbak-auart,
         vkorg TYPE vbak-vkorg,
         vtweg TYPE vbak-vtweg,
         kunnr TYPE vbak-kunnr,
       END OF ty_vbak,

       BEGIN OF ty_vbap,
         vbeln TYPE vbap-vbeln,
         posnr TYPE vbap-posnr,
         werks TYPE vbap-matnr,
         matnr TYPE vbap-matnr,
         kdmat TYPE vbap-kdmat,
       END OF ty_vbap,

       BEGIN OF ty_vbep,
         vbeln TYPE vbep-vbeln,
         posnr TYPE vbep-posnr,
         etenr TYPE vbep-etenr,
         wmeng TYPE vbep-wmeng,
         edatu TYPE vbep-edatu,
         lifsp TYPE vbep-lifsp,
       END OF ty_vbep.
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Início
*TYPES: BEGIN OF ty_mslb,
*          matnr     TYPE mslb-matnr,
*          werks     TYPE mslb-werks,
*          charg     TYPE mslb-charg,
*          lblab     TYPE mslb-lblab,
*          lifnr     TYPE mslb-lifnr,
*       END OF ty_mslb.
*DATA: t_mslb_log    TYPE TABLE OF ty_mslb,
*      wa_mslb_log   TYPE ty_mslb.
DATA: t_mslb_log   TYPE TABLE OF mslb.
DATA: wa_mslb_log  TYPE mslb.
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Fim

"Início-Insert-UZ97768-INC000000118848 30 10 2024
DATA: BEGIN OF t_mcha_mch1_mchb OCCURS 0.
DATA: mch1-matnr    TYPE mch1-matnr,
      mch1-charg    TYPE mch1-charg,
      mchb-lgort    TYPE mchb-lgort,
      mchb-werks    TYPE mchb-werks,
      vs_motivo     TYPE mchb-chjin,
      mchb-cspem    TYPE mchb-cspem,
      mchb-clabs    TYPE mchb-clabs,
      mch1-ersda    TYPE mch1-ersda,
      vs_peso_bruto TYPE mchb-cinsm,
      vi_saldo      TYPE p,
      END OF t_mcha_mch1_mchb.
"Fim-Insert-UZ97768-INC000000118848 30 10 2024

DATA: it_vbak TYPE TABLE OF ty_vbak,
      it_vbap TYPE TABLE OF ty_vbap,
      it_vbep TYPE TABLE OF ty_vbep.

DATA: wa_vbak_mapa TYPE ty_vbak,
      wa_vbap_mapa TYPE ty_vbap,
      wa_vbep_mapa TYPE ty_vbep.

DATA: BEGIN OF it_vbep_vbap_vbak OCCURS 0,
        werks LIKE vbap-werks,
        matnr LIKE vbap-matnr,
        vbeln LIKE vbep-vbeln,
        posnr LIKE vbep-posnr,
        kdmat LIKE vbap-kdmat,
        etenr LIKE vbep-etenr,
        kunnr LIKE vbak-kunnr,
      END OF it_vbep_vbap_vbak.

DATA: it_entped_it LIKE TABLE OF zsuppt_entped_it WITH HEADER LINE.

DATA: it_entped_hd LIKE TABLE OF zsuppt_entped_hd WITH HEADER LINE.

TYPES: BEGIN OF ty_eban, "REQUISIÇAO DE COMPRAS
         banfn   TYPE eban-banfn,             "Nº requisição de compra
         bnfpo   TYPE eban-bnfpo,             "Nº do item da requisição de compra
         matnr   TYPE eban-matnr,             "Nº do material
         werks   TYPE eban-werks,             "Centro
         ebeln   TYPE eban-ebeln,             "Pedido
         ebelp   TYPE eban-ebelp,             "Item
         menge   TYPE eban-menge,             "Quantidade da requisição de compra
         ebakz   TYPE eban-ebakz,             "Requisição de compra concluída
         zzvbeln TYPE eban-zzvbeln,         "OV
         zzposnr TYPE eban-zzposnr,         "Item
         meins   TYPE eban-meins,
       END OF ty_eban,

       BEGIN OF ty_ekko,
         ebeln TYPE ekko-ebeln,        "Nº do documento de compras
         bukrs TYPE ekko-bukrs,        "Empresa
         ernam TYPE ekko-ernam,        "Nome responsável que criou o objeto
         lifnr TYPE ekko-lifnr,        "Nº conta do fornecedor
         bedat TYPE ekko-bedat,        "Data do documento de compra
         ebelp TYPE ekpo-ebelp,        "Nº item do documento de compra
         matnr TYPE ekpo-matnr,        "Nº do Material
         werks TYPE ekpo-werks,        "Centro
         menge TYPE ekpo-menge,        "Quantidade do pedido
         meins TYPE ekpo-meins,        "Unidade de pedido
         netpr TYPE ekpo-netpr,        "Preço líquido no documento de compra
         brtwr TYPE ekpo-brtwr,        "Valor do pedido bruto em moeda de pedido
         anfnr TYPE ekpo-anfnr,        "Nº solicitação de cotação
         anfps TYPE ekpo-anfps,        "Nº item da solicitação de cotação
         banfn TYPE cdhdr-objectid,    "Nº requisição de compra
         bnfpo TYPE ekpo-bnfpo,        "Nº do item da requisição de compra
         name1 TYPE lfa1-name1,        "Nome 1
       END OF ty_ekko,

       BEGIN OF ty_ekko_aux,
         ebeln TYPE cdhdr-objectid,    "Nº do documento de compras
         bukrs TYPE ekko-bukrs,        "Empresa
         ernam TYPE ekko-ernam,        "Nome responsável que criou o objeto
         lifnr TYPE ekko-lifnr,        "Nº conta do fornecedor
         bedat TYPE ekko-bedat,        "Data do documento de compra
         ebelp TYPE ekpo-ebelp,        "Nº item do documento de compra
         matnr TYPE ekpo-matnr,        "Nº do Material
         werks TYPE ekpo-werks,        "Centro
         menge TYPE ekpo-menge,        "Quantidade do pedido
         meins TYPE ekpo-meins,        "Unidade de pedido
         netpr TYPE ekpo-netpr,        "Preço líquido no documento de compra
         brtwr TYPE ekpo-brtwr,        "Valor do pedido bruto em moeda de pedido
         anfnr TYPE ekpo-anfnr,        "Nº solicitação de cotação
         anfps TYPE ekpo-anfps,        "Nº item da solicitação de cotação
         banfn TYPE ekpo-banfn,        "Nº requisição de compra
         bnfpo TYPE ekpo-bnfpo,        "Nº do item da requisição de compra
         name1 TYPE lfa1-name1,        "Nome 1
       END OF ty_ekko_aux,

       BEGIN OF ty_marc,
         matnr TYPE marc-matnr,
         werks TYPE marc-werks,
         rwpro TYPE marc-rwpro,
       END OF ty_marc,

       BEGIN OF ty_t438u,
         rwpro TYPE t438u-rwpro,
         mirwx TYPE t438u-mirwx,
       END OF ty_t438u,

       BEGIN OF ty_t438r,
         rwpro TYPE t438r-rwpro,
         rw1tg TYPE t438r-rw1tg,
       END OF ty_t438r.

DATA: wa_marc  TYPE ty_marc,
      wa_t438u TYPE ty_t438u,
      wa_t438r TYPE ty_t438r.

DATA: BEGIN OF t_ekbe OCCURS 0,
        ebeln LIKE ekbe-ebeln,
        ebelp LIKE ekbe-ebelp,
        belnr LIKE ekbe-belnr,
        buzei LIKE ekbe-buzei,
        menge LIKE ekbe-menge,
        cpudt LIKE ekbe-cpudt,
        cputm LIKE ekbe-cputm,
        matnr LIKE ekbe-matnr,
        charg LIKE ekbe-charg,
      END OF t_ekbe.

DATA: t_eban      TYPE TABLE OF ty_eban,
*      t_eban_aux  TYPE TABLE OF ty_eban,     "INC 3258470
      wa_eban     TYPE ty_eban,
      t_ekko      TYPE TABLE OF ty_ekko,
      wa_ekko     TYPE ty_ekko,
      t_ekko_aux  TYPE TABLE OF ty_ekko_aux,
      wa_ekko_aux TYPE ty_ekko_aux.

DATA: BEGIN OF t_proa OCCURS 0,
        znumoa  LIKE zrnppt_proa-znumoa,
        znumpp  LIKE zrnppt_proa-znumpp,
        zoperpp LIKE zrnppt_proa-zoperpp,
        zcentr  LIKE zrnppt_proa-zcentr,
      END OF t_proa.

DATA: BEGIN OF field_definition OCCURS 0,
        fieldname    TYPE c LENGTH 30,
        outputlen    TYPE c LENGTH 6,
        inttype      TYPE c LENGTH 1,
        reptext_ddic TYPE c LENGTH 55,
      END OF field_definition.

DATA: w_field_definition LIKE LINE OF field_definition.

DATA: BEGIN OF t_definition OCCURS 0,
        fieldname    TYPE c LENGTH 30,
        outputlen    TYPE c LENGTH 6,
        reptext_ddic TYPE c LENGTH 55,
      END OF t_definition.

DATA: BEGIN OF fields OCCURS 0,
        reptext_ddic TYPE c LENGTH 55,
      END OF fields.

DATA: BEGIN OF t_saida_prev OCCURS 0,
*       sortl    LIKE kna1-sortl,
        sortl                  LIKE adrc-sort1,
        material               LIKE knmt-kdmat,
        matnr                  LIKE vbap-matnr,
        um                     LIKE vbap-meins,
        op                     TYPE vbep-wmeng, "afko-gamng,
        opa                    TYPE vbep-wmeng, "afko-gamng,
        programa               LIKE vbep-wmeng,
        faturado               LIKE vbep-wmeng,
        pendente               LIKE vbep-wmeng,
        estoque                LIKE vbep-wmeng,
***Inserção - Hermiro Carvalho - RDM 133484 - 09/03/2015 - Início
        ausch                  LIKE vbep-wmeng, "stpo-ausch,
        peso                   LIKE vbep-wmeng, "stpo-ausch,
        mat_prime              LIKE stpo-idnrk,
        estoque_mp             LIKE mchb-clabs,
        roteiro                TYPE c LENGTH 9,
        ferramenta             LIKE crvm_b-matnr,
***Inserção - Hermiro Carvalho - RDM 133484 - 09/03/2015 - Fim
        estoque_to             LIKE vbep-wmeng,
        estoque_pc             TYPE p,
        dia_01                 LIKE vbep-wmeng,
        dia_02                 LIKE vbep-wmeng,
        dia_03                 LIKE vbep-wmeng,
        dia_04                 LIKE vbep-wmeng,
        dia_05                 LIKE vbep-wmeng,
        dia_06                 LIKE vbep-wmeng,
        dia_07                 LIKE vbep-wmeng,
        dia_08                 LIKE vbep-wmeng,
        dia_09                 LIKE vbep-wmeng,
        dia_10                 LIKE vbep-wmeng,
        dia_11                 LIKE vbep-wmeng,
        dia_12                 LIKE vbep-wmeng,
        dia_13                 LIKE vbep-wmeng,
        dia_14                 LIKE vbep-wmeng,
        dia_15                 LIKE vbep-wmeng,
        dia_16                 LIKE vbep-wmeng,
        dia_17                 LIKE vbep-wmeng,
        dia_18                 LIKE vbep-wmeng,
        dia_19                 LIKE vbep-wmeng,
        dia_20                 LIKE vbep-wmeng,
        dia_21                 LIKE vbep-wmeng,
        dia_22                 LIKE vbep-wmeng,
        dia_23                 LIKE vbep-wmeng,
        dia_24                 LIKE vbep-wmeng,
        dia_25                 LIKE vbep-wmeng,
        dia_26                 LIKE vbep-wmeng,
        dia_27                 LIKE vbep-wmeng,
        dia_28                 LIKE vbep-wmeng,
        dia_29                 LIKE vbep-wmeng,
        dia_30                 LIKE vbep-wmeng,
        dia_31                 LIKE vbep-wmeng,
        mes_1                  LIKE vbep-wmeng,
        werks                  LIKE vbap-werks,
        kunnr                  LIKE zsdt_part_number-kunnr,
*       kunnr(20) TYPE c,
        projeto                LIKE zsdt_part_number-projeto,
        meses_anteriores       LIKE vbep-wmeng,
        qtde_sku               TYPE i,
        qtde_pen_fatu          TYPE i,
        indic_atend            TYPE string,
        programaateadata       TYPE vbep-wmeng,
        pendenteateadata       TYPE vbep-wmeng,
        semestoqueateadata     TYPE vbep-wmeng,
        semfaturamentoateadata TYPE vbep-wmeng,
*       programaateadatac  TYPE vbep-wmeng,
        lgpro                  TYPE marc-lgpro,
        color_cell             TYPE lvc_t_scol,    "Estrutura para colorir células
        celltab                TYPE lvc_t_styl, " cell style
      END OF  t_saida_prev.

DATA: w_definition LIKE LINE OF t_definition.

************************************************************************
**  Definição de Variáveis                                            **
************************************************************************
DATA: v_t_ord(3)            TYPE c,             " Tipo da Ordem
      v_perc                LIKE plaf-gsmng,    " percentual da tolerancia
      v_edatu               LIKE vbep-edatu,    " Data Fim da Ordem Venda
      v_centro              LIKE crhd-arbpl,    " CENTRO SEM MOVIMENTAÇÃO
      v_dat_pz              LIKE sy-datum,      "Prazo de Entrega
      v_ano_de(4)           TYPE c,
      v_ano_ate(4)          TYPE c,
      v_ok(1)               TYPE c,
      v_expedi              LIKE t_mska-kalab,
      v_expedi_aux          LIKE t_mska-kalab,
      v_exp_inter           LIKE t_mska-kalab,
      v_mat_cq              LIKE t_mska-kalab,
      v_operacao            TYPE i,
      v_wmeng               LIKE vbep-wmeng,
      v_neces_mp            LIKE vbep-wmeng,
***Inserção - Hermiro Carvalho - RDM 105472 - 27/08/2014 - Início
      v_neces_mp_aux        LIKE vbep-wmeng,
***Inserção - Hermiro Carvalho - RDM 105472 - 27/08/2014 - Fim
      v_ferramenta          LIKE crvm_b-matnr,
      v_material            LIKE afpo-matnr,
      v_vrkme               LIKE vbep-vrkme,
      v_maius(30)           TYPE c,              "Converte p/maiúsculas
      v_maquina(8)          TYPE c,
      v_qtde                TYPE p DECIMALS 5,
      v_tabix               LIKE sy-tabix,
      v_erro(1),
      v_klart               TYPE ausp-klart,
      v_instid              TYPE srgbtbrel-instid_a,
      vl_matcli(30),
      v_saldo               LIKE mchb-clabs,
      v_cq                  LIKE mchb-clabs,
      v_saldo_mp            LIKE mchb-clabs,
      v_menor_saldo         LIKE mchb-clabs,
      v_roteiro(30),
      wa_saida              LIKE t_saida,
      v_peca_expedic        LIKE zrnppt_pmvm-zqtpeca,
      v_peca_mat_cq         LIKE zrnppt_pmvm-zqtpeca,
      v_peca_exp_int        LIKE zrnppt_pmvm-zqtpeca,
      "v_cont          TYPE i,
      v_atinn_sd_peso_blank LIKE ausp-atinn,
      v_peca_expedic_aux    LIKE zrnppt_pmvm-zqtpeca,
      v_periodo             TYPE sy-datum,
      vl_obj_name           TYPE c LENGTH 10,
      v_pl                  TYPE c LENGTH 04,
      v_fornecedor          TYPE zrnppt_pl_corte-lifnr,
      v_sobsl               TYPE sobsl,
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
      v_pecas               TYPE zrnppt_prev-peca,
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim
      v_clabs               TYPE zsuppt_lista_alt-clabs.

CONSTANTS: c_30       TYPE marc-sobsl VALUE '30',
           c_werks(7) TYPE c VALUE 'S_WERKS',
           c_e        TYPE c VALUE 'E',
           c_s        TYPE c VALUE 'S',
           c_i        TYPE c VALUE 'I',
           c_x        TYPE c VALUE 'X',
           c_eq(2)    TYPE c VALUE 'EQ'.

***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
DATA: t_tabela_ov LIKE zrnmme_caractov OCCURS 0 WITH HEADER LINE.
DATA: v_atinn TYPE cabn-atinn.
DATA: v_pni   TYPE zqmt_partnumber-partnumber,
      v_kunnr TYPE vbak-kunnr,
      v_kdmat TYPE vbap-kdmat.

*RDM 190430 - 21.11.17 – Inicio
DATA: t_caractmat      LIKE zrnmme_poa_caract_ma OCCURS 0 WITH HEADER LINE,
      v_atinn_pesopeca TYPE cabn-atinn.
*RDM 190430 - 21.11.17 – Fim

***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
************************************************************************
** RANGES                                                             **
************************************************************************
RANGES: r_auart   FOR aufk-auart,
        r_mes     FOR zrnppt_prev-mes,
        r_ano     FOR zrnppt_prev-ano,
        r_bismt   FOR zrnppt_prev-bismt,
        r_lgort   FOR tvarvc-low.

***Inserção - Hermiro Carvalho - PRJ01313 - 14/09/2015 - Início
TYPES: BEGIN OF ty_mostra_lotes,
         flag         TYPE c,
         matnr_mp     TYPE char35,
         charg_mp     TYPE char35,
         peso         TYPE p DECIMALS 3,
         prod         TYPE char35,
         espessura    TYPE p DECIMALS 2,
         largura      TYPE p DECIMALS 1,
***Inserção - Hermiro Carvalho - PRJ01313 - 12/01/2016 - Início
*    comprimento  TYPE char35,
         comprimento  TYPE p DECIMALS 2,
***Inserção - Hermiro Carvalho - PRJ01313 - 12/01/2016 - Fim
         b_maior      TYPE char35,
         b_menor      TYPE char35,
         qualidade    TYPE char35,
         quali_aten   TYPE char35,
         data_estoque TYPE i,
         status       TYPE char35,
         ov_it        TYPE char35,
         partnumber   TYPE zqmt_partnumber-partnumber,
         matnr        TYPE char35,
         pecas        TYPE zrnppt_pl_itens-qtd_pecas,
       END OF ty_mostra_lotes,

       BEGIN OF ty_resb,
         charg        TYPE resb-charg,
         data_estoque TYPE i,
       END OF ty_resb,

       BEGIN OF ty_pn_mrp,
         partnumber(35) TYPE c,
         werks          TYPE zqmt_partn_mrp-werks,
         matnr          TYPE zqmt_partnumber-matnr,
       END OF ty_pn_mrp.

***Inserção - Hermiro Carvalho - PRJ01313 - 16/10/2015 - Início
DATA: BEGIN OF t_ov OCCURS 0,
        vbeln TYPE vbap-vbeln,
        posnr TYPE vbap-posnr,
      END OF t_ov,

      BEGIN OF t_matnr OCCURS 0,
        matnr TYPE mara-matnr,
      END OF t_matnr,

      BEGIN OF t_zplcorte OCCURS 0,
        num_pl_corte  TYPE zrnppt_pl_corte-num_pl_corte,
        werks         TYPE zrnppt_pl_corte-werks,
        lifnr         TYPE zrnppt_pl_corte-lifnr,
        modelo        TYPE zrnppt_pl_corte-modelo,
        separa_bobina TYPE zrnppt_pl_corte-separa_bobina,
      END OF t_zplcorte.
***Inserção - Hermiro Carvalho - PRJ01313 - 16/10/2015 - Fim

DATA: BEGIN OF t_partn_mrp OCCURS 0.
        INCLUDE STRUCTURE zqmt_partn_mrp.
        DATA:   matnr TYPE zqmt_partnumber-matnr,
        kunnr TYPE zqmt_partnumber-kunnr,
        kdmat TYPE zqmt_partnumber-kdmat.
DATA: END OF t_partn_mrp.

DATA: t_plaf_lote           TYPE TABLE OF plaf,
      t_plaf_zovs_pendentes TYPE TABLE OF plaf,
      t_zgr_std_paramtable  TYPE TABLE OF zgr_std_paramtable WITH HEADER LINE,
      t_lotes               TYPE TABLE OF zgr_std_xtsl WITH HEADER LINE,
      t_caracteristicas     TYPE TABLE OF zgr_std_xtspl WITH HEADER LINE,
      t_return              TYPE TABLE OF bapiret2 WITH HEADER LINE,
      t_propmappings        TYPE TABLE OF zgr_propmappings WITH HEADER LINE,
      t_date_range          TYPE TABLE OF date_rg WITH HEADER LINE,
      t_mostra_lotes        TYPE TABLE OF ty_mostra_lotes WITH HEADER LINE,
      t_lotes_ovs           TYPE TABLE OF ty_resb,
      t_lotes_ovs_aux       TYPE TABLE OF ty_resb,
      wa_lotes_prevp        TYPE ty_resb,
      wa_plaf_lote          TYPE plaf,
      t_partnumber          TYPE TABLE OF zqmt_partnumber WITH HEADER LINE,
      t_mrp                 TYPE TABLE OF ty_pn_mrp,
      v_vbeln               TYPE vbap-vbeln,
      v_posnr               TYPE vbap-posnr,
      v_kdmat_aux           TYPE vbap-kdmat,
      v_resp                TYPE c,
      v_date_from           TYPE sy-datum,
      v_screen              TYPE sy-dynnr,
      v_days                TYPE i.

DATA: vl_tot_neces_mp      LIKE vbep-wmeng.
***Inserção - Hermiro Carvalho - RDM 160097 - 04/05/2016 - Início
TYPES: BEGIN OF ty_melhoria,
         werks TYPE zrnppt_prev-werks,
         matnr TYPE zrnppt_prev-matnr,
         wrk02 TYPE zrnppt_prev-werks,
       END OF ty_melhoria.
DATA: t_melhoria  TYPE TABLE OF ty_melhoria,
      wa_melhoria TYPE ty_melhoria.
DATA: seltab    TYPE TABLE OF rsparams,
      seltab_wa LIKE LINE OF seltab.
***Inserção - Hermiro Carvalho - RDM 160097 - 04/05/2016 - Fim
CONTROLS: z_lotes TYPE TABLEVIEW USING SCREEN 9040.
***Inserção - Hermiro Carvalho - PRJ01313 - 14/09/2015 - Fim

* Sidney Q de Andrade - 28.03.17 - RDM 177996 - Inicio
DATA: lo_manager  TYPE REF TO cl_gos_manager,
      lt_services TYPE tgos_sels,
      ls_service  TYPE sgos_sels,
      la_obj      TYPE borident.
* Sidney Q de Andrade - 28.03.17 - RDM 177996 - Fim

TYPES: BEGIN OF ty_resb_werks,
         werks        TYPE resb-werks,
         charg        TYPE resb-charg,
         data_estoque TYPE i,
       END OF ty_resb_werks.

DATA: t_materiais_alv        TYPE zpp_mostra_lotes_alv_mat_tab,
      t_mostra_lotes_sel     TYPE zpp_mostra_lotes_alv_tab WITH HEADER LINE,
      t_ordens_find          TYPE zpp_mostra_lotes_alv_tab,
      t_lotes_ovs_werks      TYPE TABLE OF ty_resb_werks,
      t_lotes_ovs_werks_aux  TYPE TABLE OF ty_resb_werks,
      wa_mate_alv            TYPE LINE OF zpp_mostra_lotes_alv_mat_tab,
      wa_orve_find           TYPE LINE OF zpp_mostra_lotes_alv_tab,
      wa_lotes_ovs_werks     TYPE ty_resb_werks,
      wa_lotes_ovs_werks_aux TYPE ty_resb_werks,
      v_string               TYPE string.

TYPES: BEGIN OF ty_alv_popup,
         werks         TYPE crhd-werks,
         lgort         TYPE marc-lgpro,
         cliente       TYPE char10,
         cod_cli       TYPE kna1-kunnr,
         matnr         TYPE zrnppt_prev-matnr,
         materia_prima TYPE mara-matnr,
       END OF ty_alv_popup.

TYPES: BEGIN OF ty_saida_resumo,
         werks      TYPE crhd-werks,
         lgort      TYPE marc-lgpro,
         cliente    TYPE char10,
         cod_cli    TYPE kna1-kunnr,
         status_org TYPE char20,
         qtde_sku   TYPE i,
         per_orc    TYPE char04,
         s_mp       TYPE i,
         status_60d TYPE char20,
         qtde_sku60 TYPE i,
         perc_60    TYPE char05,
         cellcolor  TYPE lvc_t_scol,    "Estrutura para colorir células
       END OF ty_saida_resumo.
DATA: t_saida_resumo       TYPE TABLE OF ty_saida_resumo,
      t_saida_resumo_aux   LIKE TABLE OF t_saida,
      t_saida_resumo_aux2  LIKE TABLE OF t_saida,
      wa_saida_resumo      TYPE ty_saida_resumo,
      wa_saida_resumo_aux  LIKE LINE OF t_saida,
      wa_saida_resumo_aux2 LIKE LINE OF t_saida.

INCLUDE zrnppi_planej_prod.

*PP - Extrator dos dados rel. GD para PowerBI - RDM 246412
*-----------------------------------------------------------------------
*  Evento: AT SELECTION-SCREEN
*-----------------------------------------------------------------------
AT SELECTION-SCREEN.
  gv_ucomm = sy-ucomm.
*PP - Extrator dos dados rel. GD para PowerBI - RDM 246412

* Bruno Quadros - CSC - 12.04.2011 - Inicio
*-----------------------------------------------------------------------
*  Evento: ON SELECTION SCREEN
*-----------------------------------------------------------------------
AT SELECTION-SCREEN ON VALUE-REQUEST FOR pc_vari.
  PERFORM zf_busca_variante.
  PERFORM zf_valida_variante.
* Bruno Quadros - CSC - 12.04.2011 - Fim

AT SELECTION-SCREEN ON VALUE-REQUEST FOR p_file.
  p_file = lcl_dep=>f4( ).

*AT SELECTION-SCREEN ON VALUE-REQUEST FOR so-low.
*  PERFORM build_f4_delimiter.

AT SELECTION-SCREEN OUTPUT.
  IF p_attmod EQ 'X'.
    CLEAR: p_0001 , p_csv.

  ENDIF.
*PP - Extrator dos dados rel. GD para PowerBI - RDM 246412
  CASE gv_ucomm.
    WHEN 'PATH'.
      LOOP AT SCREEN.
        IF screen-name = 'P_FILE'.
*RDM 324238- Begin
          IF p_csv IS NOT INITIAL.
            IF p_serv IS NOT INITIAL.
              p_file = |/producao/bw/work/GD_{ sy-datum+6 }_{ sy-datum+4(2) }_{ sy-datum(4) }.csv|..
            ELSE.
              p_file = 'C:\temp\export_zpp_prevp.csv'.
            ENDIF.
          ELSE.
*RDM 324238- End
            IF p_serv IS NOT INITIAL.
              p_file = '/producao/bw/work/dwsu_gd.txt'.
            ELSE.
              p_file = 'C:\temp\dwsu_gd.txt'.
            ENDIF.
*RDM 324238- Begin
          ENDIF.
*RDM 324238- End
          MODIFY SCREEN.
        ENDIF.
        IF p_attmod EQ 'X'.
          CLEAR: p_0001 , p_csv.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    WHEN OTHERS.
*      IF p_0001 IS INITIAL.
      IF p_0001 IS INITIAL AND p_csv IS INITIAL.            "RDM 324238
        LOOP AT SCREEN.
          IF screen-group1 = 'ID1'.
            screen-active = 0.
            MODIFY SCREEN.
          ENDIF.

        ENDLOOP.
      ELSE.
        LOOP AT SCREEN.
          IF screen-group1 = 'ID1'.
            screen-active = 1.
            MODIFY SCREEN.
          ENDIF.

          IF screen-name = 'P_FILE'.
            IF p_0001 IS NOT INITIAL.                       "RDM 324238
              IF p_serv IS NOT INITIAL.
                p_file = '/producao/bw/work/dwsu_gd.txt'.
              ELSE.
                p_file = 'C:\temp\dwsu_gd.txt'.
              ENDIF.
*RDM 324238- Begin
            ELSEIF p_csv IS NOT INITIAL. "RDM
              IF p_serv IS NOT INITIAL.
                p_file = |/producao/bw/work/GD_{ sy-datum+6 }_{ sy-datum+4(2) }_{ sy-datum(4) }.csv|..
              ELSE.
                p_file = 'C:\temp\export_zpp_prevp.csv'.
              ENDIF.
            ENDIF.
          ELSEIF screen-name = 'P_0001'.
            IF p_csv IS NOT INITIAL.
              p_0001 = ''.
            ENDIF.
          ELSEIF screen-name = 'P_CSV'.
            IF p_0001 IS NOT INITIAL.
              p_csv = ''.
            ENDIF.
*RDM 324238- End
          ENDIF.

          IF p_attmod EQ 'X'.
            CLEAR: p_0001 , p_csv.
            MODIFY SCREEN.
          ENDIF.
        ENDLOOP.
      ENDIF.
*RDM 324238- Begin
*      IF p_csv IS INITIAL.
*        LOOP AT SCREEN.
*          IF screen-group1 = 'ID1'.
*            screen-active = 0.
*            MODIFY SCREEN.
*          ENDIF.
*
*        ENDLOOP.
*      ELSE.
*        LOOP AT SCREEN.
*          IF screen-group1 = 'ID1'.
*            screen-active = 1.
*            MODIFY SCREEN.
*          ENDIF.
*
*          IF screen-name = 'P_FILE'.
*            IF p_serv IS NOT INITIAL.
*              p_file = |/producao/bw/work/GD_{ sy-datum+6 }_{ sy-datum+4(2) }_{ sy-datum(4) }.csv|..
*            ELSE.
*              p_file = 'C:\temp\export_zpp_prevp.csv'.
*            ENDIF.
*          ENDIF.
*        ENDLOOP.
*    ENDIF.
*RDM 324238 - End
  ENDCASE.
*PP - Extrator dos dados rel. GD para PowerBI - RDM 246412

************************************************************************
** Inicialização                                                      **
************************************************************************
INITIALIZATION.

*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896
  CLEAR gv_program.
  IMPORT gv_program FROM MEMORY ID 'GATILHO_JIT'.
  FREE MEMORY ID 'GATILHO_JIT'.
*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896

* Bruno Quadros - CSC - 12.04.2011 - Inicio
  PERFORM zf_inicializa.    "inicializa variante
* Bruno Quadros - CSC - 12.04.2011 - Fim

  PERFORM f_inicia_range.

************************************************************************
START-OF-SELECTION.
************************************************************************

* Stefanini/Cristiane Souza – RDM 338761 - 05/11/2026 - Início
  IF p_bckgrd EQ 'X'.
    sy-tcode = 'ZPP_PREVP'.
  ENDIF.
* Stefanini/Cristiane Souza – RDM 338761 - 05/11/2026 - Fim

  IF p_period-high IS INITIAL.
    p_period-high = '99999999'.
  ENDIF.

  CLEAR: r_mes, r_mes[], r_ano, r_ano[].

  r_mes-sign   = 'I'.
  r_mes-option = 'BT'.
  r_mes-low    = p_period-low+4(2).
  r_mes-high   = p_period-high+4(2).
  APPEND r_mes.

  v_ano_de  = p_period-low(4).
  v_ano_ate = p_period-high(4).

* Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
  r_ano-sign   = 'I'.
  r_ano-option = 'BT'.
  r_ano-low    = p_period-low(4).

  IF p_period-high = '99999999'.       ">>>RDM 321837-correção seleção período
    r_ano-high    = p_period-low(4).
    p_period-high = p_period-low.      "<<<RDM 321837-correção seleção período
  ELSE.
    r_ano-high   = p_period-high(4).
  ENDIF.
  APPEND r_ano.
* Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Fim

* Alteração - Sidney Q de Andrade - SU - 03.01.2012 - Inicio
*  IF v_ano_de <> v_ano_ate AND v_ano_ate <> '0000'.
*    MESSAGE s897(sd) WITH text-008.
*    CHECK v_ano_de = v_ano_ate.
*  ENDIF.
* Alteração - Sidney Q de Andrade - SU - 03.01.2012 - Fim

* Seleciona objid
  PERFORM f_seleciona_crhd.

* Grazielle Alves - Procwork - Início - 17/09/2008 - Projeto Complementar - PP128
* Selecionar Cliente
  IF NOT p_clt IS INITIAL.
    PERFORM seleciona_cliente.
  ENDIF.
* Grazielle Alves - Procwork - Fim    - 17/09/2008 - Projeto Complementar - PP128

* INICIO: RCG8096 RDM 191755

  MOVE: c_werks    TO seltab_wa-selname,
        c_s       TO seltab_wa-kind,
        c_i       TO seltab_wa-sign,
        c_eq      TO seltab_wa-option,
        p_centro-low  TO seltab_wa-low.

  APPEND seltab_wa TO seltab.

*  SUBMIT zsuppr_mapa WITH SELECTION-TABLE seltab
*                     WITH s_matnr IN p_mat
*                     WITH p_mes EQ p_period-low+4(2)
*                     WITH p_ano EQ p_period-low(4)
*                     WITH p_todos EQ c_x AND RETURN.
*  CLEAR: t_saida_prev[].
*  REFRESH: t_saida_prev.
*
*  IMPORT gt_saida TO t_saida_prev FROM MEMORY ID 'GT_SAIDA_MAPA'.

* FIM   : RGC8096 RDM 191755

***Inserção - Hermiro Carvalho - RDM 119419 - 18/11/2014 - Início
  IF p_matpri[] IS NOT INITIAL.
    CLEAR: t_stpo2[], wa_stpo, t_mast[], wa_mast.
    "Material Configurado
    SELECT stlty stlnr idnrk
      INTO TABLE t_stpo2
      FROM stpo
     WHERE stlty =  'M'
       AND idnrk IN p_matpri.
    CASE sy-subrc.
      WHEN 0.
        SELECT matnr werks stlan stlnr stlal
          INTO TABLE t_mast
          FROM mast
          FOR ALL ENTRIES IN t_stpo2
         WHERE werks IN p_centro
           AND stlan EQ '1'
           AND stlnr EQ t_stpo2-stlnr.
      WHEN OTHERS.
    ENDCASE.

    "Material Configurável ou Configurado com PNI
    CLEAR: t_partn_mrp[].
    SELECT partnumber
      INTO CORRESPONDING FIELDS OF TABLE t_partn_mrp
      FROM zqmt_partn_mrp
     WHERE idnrk IN p_matpri AND
           werks IN p_centro.
    IF t_partn_mrp[] IS NOT INITIAL.
      LOOP AT t_partn_mrp.
        SELECT SINGLE matnr_antigo kunnr kdmat
          FROM zqmt_partnumber
          INTO (t_partn_mrp-matnr, t_partn_mrp-kunnr, t_partn_mrp-kdmat)
          WHERE partnumber = t_partn_mrp-partnumber AND
             ( datefrom <= p_period-low  AND dateto >= p_period-high ) AND
             ( status     = 'LIB' OR status = 'LIR' ).
        IF sy-subrc IS NOT INITIAL.
          DELETE t_partn_mrp INDEX sy-tabix.
        ELSE.
          MODIFY t_partn_mrp INDEX sy-tabix.
        ENDIF.
      ENDLOOP.
    ENDIF.

  ENDIF.
***Inserção - Hermiro Carvalho - RDM 119419 - 18/11/2014 - Fim

* Seleciona os dados para base do relatório
  PERFORM f_seleciona_zrnppt_prev.

* Carrega os centros de Trabalhos com a Chave de Controle = PP01]]]

* IF sy-uname NE 'RGC8096'.
  IF t_zrnppt_prev[] IS INITIAL.
*** Stefanini / Evandro Lopes - RDM 338277 - 27/10/2026 - Início
* Parametro para não interromper o processamendo de outros programas
* ao chamarem este (Ex. ZSDR_IC_ITENS_CRITICOS)
    IF p_bckgrd IS INITIAL.
*** Stefanini / Evandro Lopes - RDM 338277 - 27/10/2026 - Fim
      MESSAGE 'Dados não encontrados!' TYPE 'I'.
    ENDIF.      " Stefanini / Evandro Lopes - RDM 338277 - 27/10/2026
  ENDIF.
  CHECK NOT t_zrnppt_prev[] IS INITIAL.
* ENDIF.

* PERFORM pf_teste_desenv.

  PERFORM sel_cent_trab.

* Bruno Quadros - CSC - 25.04.2011 - Inicio
  PERFORM centro_produtor.
* Bruno Quadros - CSC - 25.04.2011 - Fim

* PERFORM monta_material.

* Leonardo Tércio - 23/09/2008 - Inicio
* Seleciona as Ordens Planejadas
  v_t_ord = 'PLA'.
  PERFORM sel_ord_planej.

* Seleciona as Ordens de Produção
  v_t_ord = 'PRO'.
  PERFORM sel_ord_prod.

* Busca cliente e insere na t_dad_rel
  PERFORM f_preenche_cliente.

* Seleciona e insere expedição
  PERFORM f_expedicao.

* Busca OV e OP e carrega a T_SAIDA
  PERFORM f_carrega_ov_op.

* Busca Requisição e Pedido de Compra e carrega a T_SAIDA
  PERFORM f_carrega_bext.

** Atribui código do cliente
* PERFORM F_COD_CLI.

* Seleciona Faturamento por Data de Remessa
  PERFORM f_faturado_periodo_remessa.

* Seleciona e insere faturado e qualifica demanda
  PERFORM f_faturado.

* Calcula pendencia e a produzir
*RDM 189985 - inicio
  IF t_saida[] IS NOT INITIAL.
    CLEAR: t_mast_log[], t_stpo_log[], wa_mast_log, wa_stpo_log.
    SELECT *
      INTO TABLE t_mast_log
      FROM mast
       FOR ALL ENTRIES IN t_saida
     WHERE matnr EQ t_saida-matnr
*       AND werks EQ t_saida-werks. "INC 3125037
       AND werks EQ t_saida-wrk02. "INC 3125037

    IF sy-subrc IS INITIAL.
      SELECT stlnr idnrk meins menge
        INTO CORRESPONDING FIELDS OF TABLE t_stpo_log
        FROM stpo
         FOR ALL ENTRIES IN t_mast_log
       WHERE stlnr = t_mast_log-stlnr.
    ENDIF.
  ENDIF.
*RDM 189985 - fim
  PERFORM f_pendecia_aproduzir.

* Calcula Produção Total
  PERFORM f_producao_total.

*** Início da Alteração - Silvia (UT00935) - 12.01.2023 - DEM:4972 / PRJ1353
  "Atualizar o relatório com as informações das Chamadas Kanban
  PERFORM f_busca_dados_chamada.
*** Fim    da Alteração - Silvia (UT00935) - 12.01.2023 - DEM:4972 / PRJ1353

* Stefanini/Cristiane Souza – RDM 338761 - 05/11/2026 - Início
  IF sy-tcode NE 'ZPP_PREVP'.
* Stefanini/Cristiane Souza – RDM 338761 - 05/11/2026 - Fim
*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896
    IF gv_program NE 'ZPPR_GATILHO_JIT_CONSIGNADO' AND sy-cprog NE 'ZRNPPR_PLANEJ_PROD_V2'.
*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896

* Exporta dados para extranet
      " RGC8096 RDM 210489 27Nov2018
*    IF p_0001 IS INITIAL.                          "RDM324238(-)
      IF p_0001 IS INITIAL AND p_csv IS INITIAL.            "RDM324238
        IF ( sy-tcode = 'ZPP_SAP_EXTRANET_TAB' OR sy-tcode = 'ZPP_SAP_EXTRANET' OR sy-tcode = 'SM37' OR sy-tcode = 'SE38' OR sy-batch IS NOT INITIAL ).
          EXPORT t_saida TO MEMORY ID 'EXTRANET'.
*GVCA20221020+ #4693-Ajuste na transação ZPP_PREVP (PRJ1353) Begin
          IF sy-tcode = 'SM37' OR sy-tcode = 'SE38' OR sy-batch EQ 'X'. "execução via JOB
            PERFORM f_grava_zsdt_zpp_prevp_h.
          ENDIF. "execução online (com display)
*GVCA20221020+ #4693-Ajuste na transação ZPP_PREVP (PRJ1353) End
          LEAVE SCREEN.
        ENDIF.
      ENDIF.

*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896
    ENDIF.
*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896
* Stefanini/Cristiane Souza – RDM 338761 - 05/11/2026 - Início
  ENDIF.
* Stefanini/Cristiane Souza – RDM 338761 - 05/11/2026 - Fim

* Monta ALV de saida
  PERFORM f_imprime_saida.

END-OF-SELECTION.
************************************************************************

*----------------------------------------------------------------------*
*       CLASS lcl_dep IMPLEMENTATION
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
CLASS lcl_dep IMPLEMENTATION.

  METHOD f4.

    CALL FUNCTION 'ZUPMMF_F4_FILE_PATH_SERV_LOCL'
      EXPORTING
        i_nome_rb_serv = c_serv
      CHANGING
        c_path         = file
      EXCEPTIONS
        local_invalido = 1
        tipo_invalido  = 2
        OTHERS         = 3.

  ENDMETHOD.                    "f4

  METHOD gera_arquivo_serv.

    DATA: vl_linha    TYPE string,
          vl_filename TYPE string.

    p_file = '/producao/bw/work/dwsu_gd.txt'.

    vl_filename = p_file.

    OPEN DATASET vl_filename IN TEXT MODE
                                  ENCODING DEFAULT
                                  FOR OUTPUT
                                  IGNORING CONVERSION ERRORS.
    IF sy-subrc IS INITIAL.
      LOOP AT t_text INTO vl_linha.
        TRANSFER vl_linha TO vl_filename.
      ENDLOOP.
      MESSAGE text-007 TYPE c_s.
    ELSE.
      MESSAGE text-008 TYPE c_s.
    ENDIF.

    CLOSE DATASET vl_filename.

  ENDMETHOD.                    "gera_arquivo_serv

ENDCLASS.                    "lcl_sliic IMPLEMENTATION



************************************************************************
**** Verifica a entrada de dados
*AT SELECTION-SCREEN.

*  Verifica o Centro é válido
*  SELECT SINGLE werks INTO t001w-werks
*  FROM t001w
*  WHERE werks in p_centro-low.
*  IF sy-subrc <> 0.
*    MESSAGE e897(sd) WITH text-010.
*  ENDIF.

************************************************************************
***               R O T I N A S   D O   P R O G R A M A              ***
************************************************************************
FORM sel_ord_planej.

* Bruno Quadros - CSC - 30.09.2010 - Início
  CLEAR: t_plaf[].

  SELECT plnum matnr plwrk gsmng pedtr kdauf kdpos meins
    INTO CORRESPONDING FIELDS OF TABLE t_plaf
    FROM plaf
    FOR ALL ENTRIES IN t_saida
    WHERE matnr = t_saida-matnr
* Sandra Hernandez - abril/2017 - RDM 177618 - Início
*          AND plwrk = t_saida-wrk02
           AND pwwrk = t_saida-wrk02
* Sandra Hernandez - abril/2017 - RDM 177618 - Fim
          AND pedtr IN p_period
      AND plscn = ''.


  LOOP AT t_plaf INTO wa_plaf.

    v_tabix = sy-tabix.

*   Recupera Ordem de Venda Item e Divisão
***Inserção - Hermiro Carvalho - RDM 1055472 - 11/07/2014 - Início
*    IF wa_plaf-kdauf IS INITIAL.
***Inserção - Hermiro Carvalho - RDM 1055472 - 11/07/2014 - Fim
    PERFORM recupera_ov.
    IF sy-subrc IS NOT INITIAL.
      DELETE t_plaf INDEX v_tabix.
      CONTINUE.
    ENDIF.

    DELETE t_imdrqx WHERE matnr NE wa_plaf-matnr.
    SORT t_imdrqx BY delkz.

    CLEAR wa_imdrqx.
    READ TABLE t_imdrqx INTO wa_imdrqx WITH KEY delkz = 'PP'
                                                BINARY SEARCH.
    IF sy-subrc IS INITIAL.
      MOVE 'PP' TO wa_plaf-delkz.
    ELSE.
      CLEAR wa_imdrqx.
      READ TABLE t_imdrqx INTO wa_imdrqx WITH KEY delkz = 'VC'
                                                  BINARY SEARCH.
      IF sy-subrc IS INITIAL.
        MOVE 'VC' TO wa_plaf-delkz.
      ELSE.
        CLEAR wa_imdrqx.
        READ TABLE t_imdrqx INTO wa_imdrqx WITH KEY delkz = 'PA'
                                                    BINARY SEARCH.
        IF sy-subrc IS INITIAL.
          MOVE 'PA' TO wa_plaf-delkz.
        ELSE.
          CLEAR wa_imdrqx.
          READ TABLE t_imdrqx INTO wa_imdrqx WITH KEY delkz = 'VW'
                                                      BINARY SEARCH.
          IF sy-subrc IS INITIAL.
            MOVE 'VW' TO wa_plaf-delkz.
          ENDIF.
        ENDIF.
      ENDIF.
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 1055472 - 11/07/2014 - Início
*    ENDIF.
***Inserção - Hermiro Carvalho - RDM 1055472 - 11/07/2014 - Fim
    IF wa_plaf-delkz EQ 'VC' OR wa_plaf-delkz EQ 'VW'.
      wa_plaf-kdauf = wa_imdrqx-delnr.
      wa_plaf-kdpos = wa_imdrqx-delps.
    ENDIF.

*    Verifica se o kdauf (Ordem de Venda é branco).
*    IF wa_plaf-kdauf IS INITIAL.
*      CONTINUE.
*    ENDIF.
***Inserção - Hermiro Carvalho - RDM 1055472 - 11/07/2014 - Início
*    IF wa_plaf-kdauf IS NOT INITIAL.
    IF wa_imdrqx-delnr IS NOT INITIAL.
***Inserção - Hermiro Carvalho - RDM 1055472 - 11/07/2014 - Fim
*    Busca dados Cabeçalho da Ordem de Venda
      PERFORM acessa_vbak.
      IF sy-subrc IS NOT INITIAL.
        DELETE t_plaf INDEX v_tabix.
        CONTINUE.
      ENDIF.

*    Busca dados item da Ordem de Venda
      PERFORM acessa_vbap.
      IF sy-subrc IS NOT INITIAL.
        DELETE t_plaf INDEX v_tabix.
        CONTINUE.
      ENDIF.

*    Busca Documento SD: status de item
      PERFORM acessa_vbup.
      IF sy-subrc IS NOT INITIAL.
        DELETE t_plaf INDEX v_tabix.
        CONTINUE.
      ENDIF.

*    Busca Documento SD: status de item
      PERFORM acessa_vbuk.
      IF sy-subrc IS NOT  INITIAL.
        DELETE t_plaf INDEX v_tabix.
        CONTINUE.
      ENDIF.

      CLEAR v_wmeng.

      PERFORM prazo_entrega.             " verifica o prazo de entrega
      wa_plaf-kwmeng = v_wmeng.
      wa_plaf-pedtr  = v_edatu.

      IF ( wa_plaf-pedtr < p_period-low  ) OR ( wa_plaf-pedtr > p_period-high ).
        PERFORM trata_peso_peca.
        CONTINUE.
      ENDIF.

      PERFORM verifica_perc.

      IF v_erro IS NOT INITIAL.
        DELETE t_plaf INDEX v_tabix.
        CONTINUE.
      ENDIF.
      wa_plaf-etenr = wa_imdrqx-delet.

    ELSE.

      IF wa_plaf-delkz NE 'VC' AND wa_plaf-delkz NE 'VW'.
*    ELSE.
        SELECT bdzei UP TO 1 ROWS
         INTO mdpb-bdzei
         FROM mdpb
         WHERE matnr    = wa_imdrqx-matnr
           AND pdatu    = wa_imdrqx-dat00
           AND vervs    = 'X'.
        ENDSELECT.

        IF sy-subrc IS INITIAL.
          wa_plaf-bdzei = mdpb-bdzei.
          wa_plaf-bstdk = wa_imdrqx-dat00.
        ENDIF.
        wa_plaf-pedtr = wa_imdrqx-dat00.

        CLEAR v_wmeng.
        PERFORM prazo_entrega_gd.

        IF ( wa_plaf-pedtr < p_period-low  ) OR ( wa_plaf-pedtr > p_period-high ).
*        DELETE WA_PLAF INDEX V_TABIX.
***Inserção - Hermiro Carvalho - 12/11/2014 - INC 1749891 - Início
          PERFORM trata_peso_peca.
***Inserção - Hermiro Carvalho - 12/11/2014 - INC 1749891 - Fim
          CONTINUE.
        ENDIF.

        wa_plaf-pedtr  = v_edatu.
        wa_plaf-kwmeng = v_wmeng.
      ENDIF.
    ENDIF.

    PERFORM trata_zprazo.

    IF NOT zrnppt_contrprog-zstatus IS INITIAL.
      DELETE t_plaf INDEX v_tabix.
      CONTINUE.
    ENDIF.
    MOVE-CORRESPONDING wa_plaf TO t_plaf.
    MODIFY t_plaf INDEX v_tabix.
  ENDLOOP.

  SORT t_plaf BY kdauf kdpos.

  LOOP AT t_plaf INTO wa_plaf.
    CLEAR t_dad_rel.
***Inserção - Hermiro Carvalho - RDM 137207 - 10/04/2015 - Início
    CLEAR: v_tabix.
    v_tabix = sy-tabix.
    PERFORM grava_dad_rel USING space.
***Inserção - Hermiro Carvalho - RDM 137207 - 10/04/2015 - Fim
  ENDLOOP.

ENDFORM.                               " SEL_ORD_PLANEJ

************************************************************************
*** Form : Sel_Cent_Trab                                             ***
*** Descrição:  Seleciona os centros de trabalho com a chave de con- ***
**              trole igual a PP01                                   ***
************************************************************************
FORM sel_cent_trab.
  DATA: tl_crtx TYPE TABLE OF crtx WITH HEADER LINE.

  CLEAR t_cent_trab.
  FREE t_cent_trab.

***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
*  SELECT objty objid arbpl steus
  SELECT objty objid arbpl werks steus
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
    INTO CORRESPONDING FIELDS OF TABLE t_cent_trab
    FROM crhd
    WHERE verwe NE '0005'
      AND werks IN p_centro
      AND steus IN r_auart.

  IF sy-subrc = 0.
*   Atualiza a tabela de Centros
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
    SORT t_cent_trab BY werks arbpl ASCENDING.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
    SELECT *
      INTO TABLE tl_crtx
      FROM crtx
      FOR ALL ENTRIES IN t_cent_trab
      WHERE objty = t_cent_trab-objty
        AND objid = t_cent_trab-objid
        AND spras = sy-langu.

    IF sy-subrc IS INITIAL.
      SORT tl_crtx BY objty objid.

      LOOP AT t_cent_trab.
        v_tabix = sy-tabix.
        READ TABLE tl_crtx WITH KEY objty = t_cent_trab-objty
                                    objid = t_cent_trab-objid
                                    BINARY SEARCH.
        IF sy-subrc = 0.
          t_cent_trab-ktext = tl_crtx-ktext.
          MODIFY t_cent_trab INDEX v_tabix.
        ENDIF.
      ENDLOOP.
    ENDIF.
  ENDIF.

ENDFORM.                               " SEL_CENT_TRAB

************************************************************************
*** Form :  Atualiz_Dad_Rel                                          ***
*** Descrição: Atualiza a tabela que consta todos os dados do relató ***
**             rio                                                   ***
************************************************************************
FORM atualiz_dad_rel USING VALUE(p_centrot) LIKE crhd-arbpl.
* Verifica se o Centro de trabalho foi preenchido
  IF ( s_cent_t-low IS INITIAL ) AND ( p_centrot <> '1' ) .
*    Verifica se o centro de trabalho é da Chave de Controle PP01
    CLEAR t_cent_trab.

    READ TABLE t_cent_trab WITH KEY arbpl = p_centrot.
*                                    BINARY SEARCH.
    IF sy-subrc IS INITIAL.
*     Grava os dados
      PERFORM grava_dad_rel USING p_centrot.
    ENDIF.
  ELSE.
    IF p_centrot EQ 'LWB' AND s_cent_t CS 'LWB'.
      PERFORM grava_dad_rel USING p_centrot.
    ELSE.
      IF p_centrot IN s_cent_t.
*       Grava os dados
        PERFORM grava_dad_rel USING p_centrot.
      ENDIF.
    ENDIF.
  ENDIF.
ENDFORM.                               " ATUALIZ_DA_REL

************************************************************************
*** Form : Grava_Dad_Rel                                             ***
*** Descrição: Grava os dados do Relatório                           ***
************************************************************************
FORM grava_dad_rel USING VALUE(p_centr_trab) TYPE any.

* Atualizando campos da tabela interna

  IF p_centr_trab EQ 'MV' AND v_operacao GT 1.
    EXIT.
  ENDIF.

  IF p_centr_trab EQ 'MV'.
    p_centr_trab = '1'.
  ENDIF.

  t_dad_rel-cn_tra  =  p_centr_trab.   "Centrol de Trabalho

  IF v_t_ord = 'PLA'.

    v_material = wa_plaf-matnr.

    t_dad_rel-qte_ov = vbap-kwmeng.
    t_dad_rel-dat_pz = wa_plaf-pedtr.   "Prazo de Entrega
    t_dad_rel-nr_ov  = wa_plaf-kdauf.   "Nr. da Ordem de Venda
    t_dad_rel-it_ov  = wa_plaf-kdpos.   "Item da Ordem de Venda
    t_dad_rel-werks  = wa_plaf-plwrk.   "Item da Ordem de Venda

* Busca código do cliente
    IF NOT t_dad_rel-nr_ov IS INITIAL.
      SELECT kunnr UP TO 1 ROWS
        INTO t_dad_rel-kunnr
        FROM vbpa
        WHERE vbeln = t_dad_rel-nr_ov
          AND ( parvw = 'G' OR parvw = 'WE' ).
      ENDSELECT.

      SELECT SINGLE sortl
        INTO t_dad_rel-sortl
        FROM kna1
        WHERE kunnr = t_dad_rel-kunnr.

    ENDIF.

    t_dad_rel-or_pla = wa_plaf-plnum.   "Nr. da Ordem Planejada
    t_dad_rel-bdzei  = wa_plaf-bdzei.   " Registro de Demanda
    t_dad_rel-matnr  = wa_plaf-matnr.   " Material
    t_dad_rel-delkz  = wa_plaf-delkz.   " Tipo de Ordem Planejada
    t_dad_rel-qte_pl = wa_plaf-gsmng.   " Qte da Ordem Planejada

    IF  wa_plaf-matnr > '84999999' AND wa_plaf-matnr < '87000000'.
      t_dad_rel-tipo = 'S'.
    ELSE.
      t_dad_rel-tipo = 'V'.
    ENDIF.

    IF wa_plaf-delkz = 'VC' OR wa_plaf-delkz EQ 'VW'.
      wa_imdrqx-delnr = wa_plaf-kdauf.
      wa_imdrqx-delps = wa_plaf-kdpos.
      wa_imdrqx-delet = wa_plaf-etenr.
      PERFORM prazo_entrega.             " verifica o prazo de entrega
      t_dad_rel-dat_pz = v_edatu.
    ENDIF.

* Com ordens de produção
  ELSEIF v_t_ord = 'PRO'.

    v_material = t_aufk-matnr.

    t_dad_rel-nr_ov = t_aufk-kdauf.   "Nr. da Ordem de Venda
    t_dad_rel-it_ov = t_aufk-kdpos.   "Item da Ordem de Venda

    IF NOT t_dad_rel-nr_ov IS INITIAL.

      SELECT kunnr UP TO 1 ROWS
        FROM vbpa
        INTO t_dad_rel-kunnr
        WHERE vbeln = t_dad_rel-nr_ov
          AND ( parvw = 'G' OR parvw = 'WE' ).
      ENDSELECT.

      SELECT SINGLE sortl
        INTO t_dad_rel-sortl
        FROM kna1
        WHERE kunnr = t_dad_rel-kunnr.

    ENDIF.

    t_dad_rel-or_pro = t_aufk-aufnr.   "Nr. da Ordem Produção
    t_dad_rel-matnr  = t_aufk-matnr.

    IF t_aufk-gmein = 'KG'.
      t_dad_rel-qte_or = t_aufk-gamng / 1000. "Qte da Ordem Produção
    ELSE.
      t_dad_rel-qte_or = t_aufk-gamng.
    ENDIF.

    IF t_aufk-matnr > '84999999' AND t_aufk-matnr < '87000000'.
      t_dad_rel-tipo = 'S'.
    ELSE.
      t_dad_rel-tipo = 'V'.
    ENDIF.
***Comentar - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Início
*    t_dad_rel-cn_tra = t_aufk-werks.
    t_dad_rel-werks = t_aufk-werks.
***Comentar - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Fim

*** Início - Inclusão - Sandra Hernandez - maio/2017 = RDM 177618
    READ TABLE t_saida WITH KEY matnr = t_aufk-matnr
                                wrk02 = t_aufk-werks.
    IF sy-subrc EQ 0.
      t_dad_rel-werks  = t_saida-werks.
      t_dad_rel-werks2 = t_saida-wrk02.
    ENDIF.
*** Fim - Inclusão - Sandra Hernandez - maio/2017 = RDM 177618

***INC 2232682 - Inicio
    v_wmeng = t_dad_rel-qte_or.
***INC 2232682 - Fim

    PERFORM verifica_perc_ordem.

    IF NOT v_erro IS INITIAL.
      EXIT.
    ENDIF.

  ENDIF.

  SELECT SINGLE kdmat
    FROM vbap
    INTO t_dad_rel-kdmat
***Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Início
*    WHERE vbeln = t_dad_rel-it_ov AND
     WHERE vbeln = t_dad_rel-nr_ov AND
***Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Fim
          posnr = t_dad_rel-it_ov.

* Grava os dados
  IF v_t_ord = 'PLA'.
    IF  t_dad_rel-nr_ov > 0 AND t_dad_rel-it_ov > 0 AND t_dad_rel-or_pla > 0.
      APPEND t_dad_rel.
      CLEAR  t_dad_rel.
    ELSE.
      APPEND t_dad_rel.
      CLEAR  t_dad_rel.
    ENDIF.
  ELSEIF v_t_ord = 'PRO'.
    v_dat_pz = t_dad_rel-dat_pz.

    IF  t_dad_rel-nr_ov > 0 AND t_dad_rel-it_ov > 0.
      APPEND t_dad_rel.
      CLEAR  t_dad_rel.
    ELSE.
      APPEND t_dad_rel.
      CLEAR  t_dad_rel.
    ENDIF.
    t_dad_rel-dat_pz = v_dat_pz.
  ENDIF.

  CLEAR: v_ferramenta.
ENDFORM.                               " GRAVA_DAD_REL

************************************************************************
*** Form : Sel_Ord_Prod                                              ***
*** Descrição: Seleciona as Ordens de Produção                       ***
************************************************************************
FORM sel_ord_prod.

  DATA: BEGIN OF tl_pmvm OCCURS 0,
          znumpp  LIKE zrnppt_pmvm-znumpp,
          zcharg  LIKE zrnppt_pmvm-zcharg,
          zqtreg  LIKE zrnppt_pmvm-zqtreal,
          zqtreal LIKE zrnppt_pmvm-zqtreal,
          kunnr   LIKE zrnppt_pmvm-zcodcliente,
        END OF tl_pmvm.

* Bruno Quadros - CSC - 13.04.2011 - Inicio
*  IF t_saida[] IS NOT INITIAL.
  IF t_melhoria[] IS NOT INITIAL.

    CLEAR: t_aufk[].
*    SELECT c~aufnr c~auart c~kdauf c~kdpos c~objnr
*           c~gltrp c~gstrs c~aufpl c~gamng c~gmein o~matnr c~cy_seqnr c~werks
*      INTO TABLE t_aufk
*      FROM caufv AS c INNER JOIN afpo AS o
*      ON ( c~aufnr = o~aufnr )
****Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Início
**      FOR ALL ENTRIES IN t_saida
*      FOR ALL ENTRIES IN t_melhoria
*      WHERE o~matnr = t_melhoria-matnr
*        AND c~auart  IN r_auart
**        AND c~werks  = t_saida-wrk02
*        AND c~werks  = t_melhoria-werks
****Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Fim
*        AND c~idat2  = '0000000000'
*        AND c~getri  = '0000000000'.
***Inserção - Hermiro Carvalho - INC 2193345 - 31/08/2016 - Início
*    SELECT aufnr auart kdauf kdpos objnr
*           gltrp gstrs aufpl gamng gmein plnbez cy_seqnr werks
*      INTO TABLE t_aufk
*      FROM caufv
*      FOR ALL ENTRIES IN t_melhoria
*      WHERE auart  IN r_auart
*        AND werks  = t_melhoria-werks
*        AND idat2  = '0000000000'
*****Inserção - Sidney Q de Andrade - RDM 165623  - 11/07/2016 - Inicio
**        AND getri  = '0000000000'
*****Inserção - Sidney Q de Andrade - RDM 165623  - 11/07/2016 - Fim
*        AND plnbez = t_melhoria-matnr.
    SELECT c~aufnr c~auart c~kdauf c~kdpos c~objnr
           c~gltrp c~gstrs c~aufpl c~gamng c~gmein o~matnr c~cy_seqnr c~werks
      INTO TABLE t_aufk
      FROM caufv AS c INNER JOIN afpo AS o
      ON ( c~aufnr = o~aufnr )
      FOR ALL ENTRIES IN t_melhoria
      WHERE o~matnr = t_melhoria-matnr
        AND c~auart  IN r_auart
*****Inserção - Sandra Hernandez - RDM 177618 - abril/2017 - Inicio
*        AND c~werks  = t_melhoria-werks
*        AND c~werks  = t_melhoria-wrk02    "RDM 190430 - 17.11.17
*****Inserção - Sandra Hernandez - RDM 177618 - abril/2017 - Fim
        AND c~idat2  = '0000000000'.

***Inserção - Hermiro Carvalho - INC 2193345 - 31/08/2016 - Fim
    IF NOT t_aufk[] IS INITIAL.
**     Leandro - Projeto Unir2
**     Tirar os zeros à esquerda
*      CALL FUNCTION 'CONVERSION_EXIT_ALPHA_OUTPUT'
*        EXPORTING
*          input  = t_aufk-cy_seqnr
*        IMPORTING
*          output = t_aufk-cy_seqnr.

* Bruno Quadros - CSC - 27.04.2011 - Inicio
      t_aufk_aux[] = t_aufk[].

      DELETE t_aufk_aux WHERE cy_seqnr IS INITIAL.

      LOOP AT t_aufk_aux.
        t_aufk_aux-znumoa = t_aufk_aux-cy_seqnr.
        MODIFY t_aufk_aux INDEX sy-tabix.
      ENDLOOP.

* Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
      "Verificar se a OP foi cancelada
      IF t_aufk_aux[] IS NOT INITIAL.
        SELECT *
          INTO TABLE tl_phs_canc
          FROM zrnppt_phs
          FOR ALL ENTRIES IN t_aufk_aux
          WHERE znumoa  = t_aufk_aux-znumoa
            AND zstatus = 'CANC'.

        "Deletar as Ordens Agrupadoras com status cancelado
        IF tl_phs_canc[] IS NOT INITIAL.
          SORT t_aufk BY cy_seqnr.
          SORT t_aufk_aux BY znumoa.

          LOOP AT tl_phs_canc.

            "Limpeza T_AUFK
            READ TABLE t_aufk WITH KEY cy_seqnr = tl_phs_canc-znumoa
                                       BINARY SEARCH.
            IF sy-subrc IS INITIAL.
              LOOP AT t_aufk FROM sy-tabix.
                IF t_aufk-cy_seqnr = tl_phs_canc-znumoa.
                  DELETE t_aufk INDEX sy-tabix.
                ELSE.
                  EXIT.
                ENDIF.
              ENDLOOP.
            ENDIF.

            "Limpeza T_AUFK_AUX
            READ TABLE t_aufk_aux WITH KEY znumoa = tl_phs_canc-znumoa
                                       BINARY SEARCH.
            IF sy-subrc IS INITIAL.
              LOOP AT t_aufk_aux FROM sy-tabix.
                IF t_aufk_aux-znumoa = tl_phs_canc-znumoa.
                  DELETE t_aufk_aux INDEX sy-tabix.
                ELSE.
                  EXIT.
                ENDIF.
              ENDLOOP.
            ENDIF.

          ENDLOOP.
        ENDIF.

      ENDIF.
* Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Fim

      IF t_aufk_aux[] IS NOT INITIAL.
        SELECT znumpp zcharg zqtreg zqtreal zcodcliente
          INTO TABLE tl_pmvm
          FROM zrnppt_pmvm
          FOR ALL ENTRIES IN t_aufk_aux
          WHERE znumoa = t_aufk_aux-znumoa
            AND znumpp = t_aufk_aux-aufnr
            AND zmatnr = t_aufk_aux-matnr
            AND zcharg <> ' '
            AND zcodmov = '101'
            AND zflag <> 'X'.
      ENDIF.

      IF NOT tl_pmvm[] IS INITIAL.
        SORT tl_pmvm BY znumpp.
      ENDIF.

    ENDIF.

  ENDIF.

  SORT t_aufk BY aufnr.

  LOOP AT t_aufk.
    v_tabix = sy-tabix.

    LOOP AT tl_pmvm WHERE znumpp = t_aufk-aufnr.

      IF NOT tl_pmvm-zqtreal IS INITIAL.
        SUBTRACT tl_pmvm-zqtreal FROM t_aufk-gamng.
      ELSE.
        SUBTRACT tl_pmvm-zqtreg  FROM t_aufk-gamng.
      ENDIF.

    ENDLOOP.

    MODIFY t_aufk INDEX v_tabix.

    IF t_aufk-kdauf <> space.
      CLEAR vbup-gbsta.
      SELECT SINGLE gbsta
        INTO vbup-gbsta
        FROM vbup
        WHERE vbeln = t_aufk-kdauf
          AND posnr = t_aufk-kdpos.
      IF sy-subrc EQ 0.
        IF vbup-gbsta EQ 'C'.
          DELETE t_aufk INDEX v_tabix.
          CONTINUE.
        ENDIF.
      ENDIF.
      PERFORM trata_ov.
      IF NOT v_erro IS INITIAL.
        CONTINUE.
      ENDIF.
      MODIFY t_aufk INDEX v_tabix.
    ENDIF.

  ENDLOOP.

  SORT t_aufk BY aufnr.

  LOOP AT t_aufk.
***Inserção - Hermiro Carvalho - INC 1907528 - 27/05/2015 - Início
    v_tabix = sy-tabix.
***Inserção - Hermiro Carvalho - INC 1907528 - 27/05/2015 - Fim
    CLEAR t_dad_rel.
    t_dad_rel-dat_pz = t_aufk-gltrp.

*   Verifica se a Ordem não está agrupada
* Sandra H. - 22/09
* PERFORM veri_ord_agrup.

* verifica movimentação
    AT NEW aufnr.
      t_aufk_aux[] = t_aufk[].
      PERFORM verifica_movimentacao USING t_aufk-aufnr.
    ENDAT.
*   Lê as operações da Ordem de Produção
    PERFORM le_op_ordem.
*   Limpa a reader
    CLEAR: t_aufk, v_erro.
  ENDLOOP.
  CLEAR: v_dat_pz.
ENDFORM.                               " SEL_ORD_PROD

************************************************************************
*** Form : Le_Op_Ordem                                               ***
*** Descrição: Lê as operações da Ordem de Produção                  ***
************************************************************************
FORM le_op_ordem.

*      Verifica se o centro de Trabalho foi informado
  IF s_cent_t[] IS INITIAL.

*      Verifica se o centro é da Chave de Controle PP01
*** Sandra Hernandez - ABril / 2017 -  RDM 177618 - Início
*    CLEAR t_cent_trab.
****Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
**    READ TABLE t_cent_trab WITH KEY arbpl = crhd-arbpl
*    READ TABLE t_cent_trab WITH KEY werks = t_aufk-werks
*                                    arbpl = crhd-arbpl.
****Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
**                                    BINARY SEARCH.
*    IF sy-subrc = 0.
*** Sandra Hernandez - ABril / 2017 -  RDM 177618 - Fim

*      Grava os dados da ordem de produção
    PERFORM grava_dad_rel USING crhd-arbpl.
*   ENDIF. " RDM 177618 - Fim
  ELSE.

*         Verifica se o centro consta na entrada de dados
    SORT t_zrnppt_prev BY matnr.
    CLEAR t_zrnppt_prev.
    READ TABLE t_zrnppt_prev WITH KEY matnr = t_aufk-matnr
                                      BINARY SEARCH.

    IF t_zrnppt_prev-op1 IN s_cent_t OR
       t_zrnppt_prev-op2 IN s_cent_t OR
       t_zrnppt_prev-op3 IN s_cent_t OR
       t_zrnppt_prev-op4 IN s_cent_t OR
       t_zrnppt_prev-op5 IN s_cent_t OR
       t_zrnppt_prev-op6 IN s_cent_t OR
       t_zrnppt_prev-op7 IN s_cent_t OR
       t_zrnppt_prev-op8 IN s_cent_t.
*            Grava os dados da ordem de produção
      PERFORM grava_dad_rel  USING crhd-arbpl.
    ENDIF.
*    ENDIF.
  ENDIF.
ENDFORM.                               " LE_OP_ORDEM

************************************************************************
**&      Form  prazo de entrega
************************************************************************
FORM prazo_entrega.


***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*  DATA: v_kdmat TYPE vbap-kdmat VALUE IS INITIAL.
*
*  SELECT SINGLE kdmat
*    FROM vbap
*    INTO v_kdmat
*    WHERE vbeln = wa_imdrqx-delnr AND
*          posnr = wa_imdrqx-delps.

  CLEAR: v_pni, v_vbeln, v_posnr, t_tabela_ov[].

  v_vbeln = wa_imdrqx-delnr.
  v_posnr = wa_imdrqx-delps.

  IF v_vbeln IS NOT INITIAL.

    CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
      EXPORTING
        input  = ' PDR_PARTNUMBER_INT'
      IMPORTING
        output = v_atinn.

    CALL FUNCTION 'ZRNSDF_SEARCH_COVD_MAE'
      EXPORTING
        ordem_venda              = v_vbeln
        item_ordem               = v_posnr
        idioma                   = sy-langu
        caracteristica           = v_atinn
      TABLES
        itab_ov                  = t_tabela_ov
      EXCEPTIONS
        not_found                = 1
        caracteristica_not_found = 2
        ordem_venda_is_missing   = 3
        idioma_is_missing        = 4
        OTHERS                   = 5.
    IF sy-subrc EQ 0.

      READ TABLE t_tabela_ov INDEX 1.
      v_pni = t_tabela_ov-resul.
      IF v_pni = 'X'.
        CLEAR v_pni.
      ENDIF.
    ENDIF.

  ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim

  SELECT SINGLE edatu wmeng vrkme
    INTO (v_edatu, v_wmeng, v_vrkme)
    FROM vbep
    WHERE vbeln = wa_imdrqx-delnr
      AND posnr = wa_imdrqx-delps
      AND etenr = wa_imdrqx-delet.

* Se encontrou move para tabela nos campos dataDE e dataATE
  IF sy-subrc IS INITIAL.

    IF v_vrkme = 'KG'.
      v_wmeng  =  v_wmeng / 1000.      " Qte da Ordem Plan.
    ELSE.
      IF v_vrkme = 'ST' OR v_vrkme = 'PC'.
        PERFORM trata_peca USING wa_plaf-matnr    "#1
                                 v_pni.
        v_wmeng  =  v_qtde * v_wmeng.
      ENDIF.
    ENDIF.

  ELSE.

    v_edatu = wa_plaf-pedtr.
    IF wa_plaf-meins = 'KG'.
      v_wmeng  =  wa_plaf-kwmeng / 1000.  " Qte da Ordem Plan.
    ELSE.
      IF wa_plaf-meins = 'ST' OR wa_plaf-meins = 'PC'.
        PERFORM trata_peca USING wa_plaf-matnr      "#2
                                 v_pni.
        v_wmeng  =  v_qtde * wa_plaf-kwmeng.
      ELSE.
        v_wmeng   =  wa_plaf-kwmeng.
      ENDIF.
    ENDIF.

  ENDIF.

ENDFORM.                               " PRAZO_ENTREGA

*&---------------------------------------------------------------------*
*&      Form  VERIFICA_PERC
*&---------------------------------------------------------------------*
FORM verifica_perc.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*  DATA: v_kdmat TYPE vbap-kdmat VALUE IS INITIAL.
*  SELECT SINGLE kdmat
*    FROM vbap
*    INTO v_kdmat
*    WHERE vbeln = wa_plaf-kdauf AND
*          posnr = wa_plaf-kdpos.

  CLEAR: v_pni, t_tabela_ov[].

  IF wa_plaf-kdauf IS NOT INITIAL.

    CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
      EXPORTING
        input  = ' PDR_PARTNUMBER_INT'
      IMPORTING
        output = v_atinn.

    CALL FUNCTION 'ZRNSDF_SEARCH_COVD_MAE'
      EXPORTING
        ordem_venda              = wa_plaf-kdauf
        item_ordem               = wa_plaf-kdpos
        idioma                   = sy-langu
        caracteristica           = v_atinn
      TABLES
        itab_ov                  = t_tabela_ov
      EXCEPTIONS
        not_found                = 1
        caracteristica_not_found = 2
        ordem_venda_is_missing   = 3
        idioma_is_missing        = 4
        OTHERS                   = 5.
    IF sy-subrc EQ 0.

      READ TABLE t_tabela_ov INDEX 1.
      v_pni = t_tabela_ov-resul.
      IF v_pni = 'X'.
        CLEAR v_pni.
      ENDIF.
    ENDIF.

  ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim

  CLEAR v_erro.
* calcular e testar a tolerância do peso da divisão de remessa
  v_perc = ( wa_plaf-untto * v_wmeng ) / 100.

  IF wa_plaf-meins = 'KG'.
    wa_plaf-gsmng  =  wa_plaf-gsmng / 1000. " Qte da Ordem Plan.
  ELSE.
    IF wa_plaf-meins = 'ST' OR wa_plaf-meins = 'PC'.
      PERFORM trata_peca USING wa_plaf-matnr    "#3
                               v_pni.
      wa_plaf-gsmng =  v_qtde * wa_plaf-gsmng.
    ENDIF.
  ENDIF.

  IF wa_plaf-gsmng <= v_perc.
    v_erro = 'X'.
  ENDIF.

ENDFORM.                               " VERIFICA_PERC

*&---------------------------------------------------------------------*
*&      Form  MONTA_MATERIAL
*&---------------------------------------------------------------------*
FORM monta_material.
  SELECT matnr mtart
    INTO TABLE t_tip_mat
    FROM mara
    WHERE ( mtart = 'FERT'  OR  mtart = 'ZALT'  OR
            mtart = 'ZALP'  OR  mtart = 'ZALE'  OR
            mtart = 'ZALC'  OR  mtart = 'ZACT'  OR
            mtart = 'ZAPT'  OR  mtart = 'ZFER'  OR
* Fabio Silva - CSC - 06.10.2009 - INÍCIO
            mtart = 'ZFEJ'  OR  mtart = 'ZALJ' ).
* Fabio Silva - CSC - 06.10.2009 - FIM
*    Classifica a tabela interna
  SORT t_tip_mat BY matnr ASCENDING.
ENDFORM.                               " MONTA_MATERIAL

*&---------------------------------------------------------------------*
*&      Form  TRATA_ZPRAZO
*&---------------------------------------------------------------------*
FORM trata_zprazo.
* Modificado Por Claudio Morales 08/03/07 - Motivo : EF V02

  CLEAR zrnppt_contrprog-zstatus.
  IF wa_plaf-delkz EQ 'VC' OR wa_plaf-delkz EQ 'VW'.
    SELECT zstatus UP TO 1 ROWS
      INTO zrnppt_contrprog-zstatus
      FROM zrnppt_contrprog
      WHERE znumov  = wa_plaf-kdauf
        AND zitemov = wa_plaf-kdpos.
    ENDSELECT.
  ELSE.
    SELECT zstatus UP TO 1 ROWS
      INTO zrnppt_contrprog-zstatus
      FROM zrnppt_contrprog
      WHERE bdzei   = wa_plaf-bdzei
        AND matnr   = wa_plaf-matnr
        AND bstdk   = wa_plaf-bstdk.
    ENDSELECT.
  ENDIF.

ENDFORM.                               " TRATA_ZPRAZO

*&---------------------------------------------------------------------*
*&      Form  TRATA_LWB
*&---------------------------------------------------------------------*
FORM trata_lwb.

  CLEAR: zrnppt_dpr.
  SELECT SINGLE b~v_solex b~v_half
    INTO (zrnppt_dpr-v_solex, zrnppt_dpr-v_half)
    FROM zrnppt_dpo AS a INNER JOIN zrnppt_dpr AS b
      ON a~numdiar = b~numdiar
  WHERE  a~numdiar = b~numdiar
    AND  a~aufnr   = t_aufk-aufnr
    AND  b~werks   = t_aufk-werks.

  IF NOT zrnppt_dpr-v_solex IS INITIAL.
    t_dad_rel-cn_tra  =  'SOLEX'.      "Centrol de Trabalho
  ENDIF.

  IF NOT zrnppt_dpr-v_half IS INITIAL.
    t_dad_rel-cn_tra  =  'HALF'.       "Centrol de Trabalho
  ENDIF.

ENDFORM.                               " TRATA_LWB

*&---------------------------------------------------------------------*
*&      Form  DE_PARA_MAQUINA
*&---------------------------------------------------------------------*
FORM de_para_maquina CHANGING p_maquina.

  TRANSLATE p_maquina TO UPPER CASE.

  CASE p_maquina.
    WHEN 'OXICORTE'.
      MOVE 'X' TO p_maquina.
    WHEN 'LAVADORA'.
      MOVE 'L' TO p_maquina.
    WHEN 'GUIL FOB'.
      MOVE 'HF' TO p_maquina.
    WHEN 'GUIL N1'.
      MOVE 'H1' TO p_maquina.
    WHEN 'GUIL N2'.
      MOVE 'H2' TO p_maquina.
    WHEN 'GUIL PR'.
      MOVE 'HP' TO p_maquina.
    WHEN 'GUIL RN'.
      MOVE 'RN' TO p_maquina.
    WHEN 'TR2'.
      MOVE 'R2' TO p_maquina.
    WHEN 'TR4'.
      MOVE 'R4' TO p_maquina.
    WHEN 'TR5'.
      MOVE 'R5' TO p_maquina.
    WHEN 'TR6'.
      MOVE 'R6' TO p_maquina.
    WHEN OTHERS.
  ENDCASE.

ENDFORM.                               " DE_PARA_MAQUINA

*&---------------------------------------------------------------------*
*&      Form  TRATA_PECA
*&---------------------------------------------------------------------*
FORM trata_peca USING VALUE(p_matnr) LIKE vbap-matnr
                      VALUE(p_partnumber) LIKE zqmt_partnumber-partnumber.

  CLEAR: v_qtde.

  READ TABLE t_mara WITH KEY matnr = p_matnr
                             BINARY SEARCH.
  IF sy-subrc IS INITIAL.
    IF t_mara-kzkfg IS INITIAL.
      IF t_mara-meins = 'ST' OR t_mara-meins = 'PC'.

        READ TABLE t_marm WITH KEY matnr = p_matnr
                                   meinh = 'TO'
                                   BINARY SEARCH.
        IF sy-subrc IS INITIAL.
          IF NOT t_marm-umrez IS INITIAL.
            v_qtde = t_marm-umren / t_marm-umrez.
          ENDIF.
        ENDIF.
      ELSE.

        READ TABLE t_marm WITH KEY matnr = p_matnr
                                   meinh = 'PC'
                                   BINARY SEARCH.
        IF sy-subrc IS NOT INITIAL.
          READ TABLE t_marm WITH KEY matnr = p_matnr
                                     meinh = 'ST'
                                     BINARY SEARCH.
        ENDIF.

        IF sy-subrc IS INITIAL.
          IF NOT t_marm-umren IS INITIAL.
            v_qtde = t_marm-umrez / t_marm-umren.
          ENDIF.
        ENDIF.
      ENDIF.

    ELSE.

      DATA: lv_atinn LIKE cabn-atinn.
      CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
        EXPORTING
          input  = 'SD_PESO_BLANK'
        IMPORTING
          output = lv_atinn.

      SELECT *
        FROM zqmt_partnumber
        INTO TABLE t_partnumber
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*        WHERE kdmat = p_kdmat          AND
         WHERE partnumber = p_partnumber AND
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
              atinn      = lv_atinn    AND
              ( datefrom <= p_period-low  AND dateto >= p_period-high ) AND
              ( status     = 'LIB' OR status = 'LIR' ).
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Início
      IF sy-subrc IS INITIAL.
        READ TABLE t_partnumber INDEX 1.
        v_qtde = t_partnumber-resul / 1000.
      ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Fim
    ENDIF.
  ENDIF.

ENDFORM.                               " TRATA_PECA

*&---------------------------------------------------------------------*
*&      Form  VERIFICA_MOVIMENTAÇÃO
*&---------------------------------------------------------------------*
FORM verifica_movimentacao USING p_value.

  CLEAR: afvc, v_operacao.
  DELETE t_aufk_aux WHERE aufnr <> p_value.
  FREE t_operacao.

  LOOP AT t_aufk_aux.

    SELECT steus arbid
      INTO (afvc-steus, afvc-arbid)
      FROM afvc
     WHERE aufpl = t_aufk_aux-aufpl AND
           steus IN r_auart.

      CLEAR crhd.
      SELECT arbpl
        INTO crhd-arbpl
        FROM crhd UP TO 1 ROWS
* Projeto Unir2 - Fabiana - 28.08.2009 - Início
*        WHERE objid = afvc-arbid.
        WHERE objid = afvc-arbid
          AND verwe <> '0005'.
* Projeto Unir2 - Fabiana - 28.08.2009 - Fim
      ENDSELECT.
      t_operacao-steus = afvc-steus.
      t_operacao-arbpl = crhd-arbpl.
      APPEND t_operacao.
      CLEAR t_operacao.
    ENDSELECT.
  ENDLOOP.

  LOOP AT t_operacao WHERE steus = 'PP01'.
    ADD 1 TO v_operacao.
  ENDLOOP.

ENDFORM.                               " VERIFICA_MOVIMENTAÇÃO

*&---------------------------------------------------------------------*
*&      Form  VERI_ORD_AGRUP
*&---------------------------------------------------------------------*
FORM veri_ord_agrup.
  CLEAR: zrnppt_popa, vbep, v_wmeng.

  SELECT znumpp znumoa znumov zitemov zdivitem
  INTO  (zrnppt_popa-znumpp, zrnppt_popa-znumoa,
         zrnppt_popa-znumov, zrnppt_popa-zitemov, zrnppt_popa-zdivitem)
  FROM   zrnppt_popa UP TO 1 ROWS
  WHERE  znumpp = t_aufk-aufnr.
  ENDSELECT.

  IF sy-subrc = 0.
    SELECT SINGLE edatu wmeng
           INTO (vbep-edatu, vbep-wmeng)
           FROM vbep
           WHERE vbeln = zrnppt_popa-znumov
             AND posnr = zrnppt_popa-zitemov
             AND etenr = zrnppt_popa-zdivitem.
* Se encontrou move para tabela nos campos dataDE e dataATE
    IF sy-subrc = 0.
      t_dad_rel-dat_pz = vbep-edatu.
      v_wmeng = vbep-wmeng.
    ELSE.
      v_wmeng = t_aufk-qte_ov.
    ENDIF.

  ENDIF.

ENDFORM.                    " VERI_ORD_AGRUP

*&---------------------------------------------------------------------*
*&      Form  TRATA_OV
*&---------------------------------------------------------------------*
FORM trata_ov.
  SELECT SINGLE kwmeng untto
  INTO (t_aufk-qte_ov, t_aufk-untto)
  FROM vbap
  WHERE vbeln = t_aufk-kdauf  AND
        posnr = t_aufk-kdpos.

  IF sy-subrc = 0.
    IF vbap-abgru NE ' '.
      v_erro = 'X'.
      DELETE t_aufk INDEX v_tabix.
      EXIT.
    ENDIF.
  ENDIF.
ENDFORM.                    " TRATA_OV

*&---------------------------------------------------------------------*
*&      Form  VERIFICA_PERC_ORDEM
*&---------------------------------------------------------------------*
FORM verifica_perc_ordem.

***INC 2232682 - Inicio
  CLEAR v_erro.
***INC 2232682 - Fim

* calcular e testar a tolerância do peso da divisão de remessa
  v_perc = ( t_aufk-untto * v_wmeng ) / 100.

  IF t_aufk-gmein = 'KG'.
    t_aufk-gamng  =  t_aufk-gamng / 1000.  " Qte da Ordem
  ENDIF.

  IF t_aufk-gamng <= v_perc.
    DELETE t_aufk INDEX v_tabix.
    v_erro = 'X'.
    EXIT.
  ENDIF.

ENDFORM.                    " VERIFICA_PERC_ORDEM

*&---------------------------------------------------------------------*
*&      Form  ATRICARA
*&---------------------------------------------------------------------*
FORM atricara  USING    p_atnam LIKE  cabn-atnam
               CHANGING p_atinn LIKE cabn-atinn.

* Busca o nº de cada característica na tabela CABN e retorna na variável
  SELECT SINGLE atinn
    INTO p_atinn
    FROM cabn
    WHERE atnam = p_atnam.

ENDFORM.                    " ATRICARA

*&---------------------------------------------------------------------*
*&      Form  RECUPERA_OV
*&---------------------------------------------------------------------*
FORM recupera_ov .

  DATA: c_edelkz LIKE mdps-delkz VALUE 'PA',
        v_edelnr LIKE mdps-del12.

  CLEAR: t_emdpsx, t_imdrqx.
  FREE:  t_emdpsx, t_imdrqx.
***Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
*  v_edelnr = t_plaf-plnum.
  v_edelnr = wa_plaf-plnum.
***Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim

  CALL FUNCTION 'MD_PEGGING_NODIALOG'
    EXPORTING
      edelkz                = c_edelkz
      edelnr                = v_edelnr
      ematnr                = wa_plaf-matnr
      ewerks                = wa_plaf-plwrk
      eplwrk                = wa_plaf-plwrk
    TABLES
      emdpsx                = t_emdpsx
      imdrqx                = t_imdrqx
    EXCEPTIONS
      error                 = 1
      no_requirements_found = 2
      order_not_found       = 3
      OTHERS                = 4.

ENDFORM.                    " RECUPERA_OV

*&---------------------------------------------------------------------*
*&      Form  ACESSA_VBAK
*&---------------------------------------------------------------------*
FORM acessa_vbak .
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
  IF wa_imdrqx-delnr EQ ''.
    SELECT SINGLE erdat
    INTO wa_plaf-erdat
    FROM vbak
    WHERE vbeln = wa_plaf-kdauf.

  ELSE.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim
    SELECT SINGLE erdat
      INTO wa_plaf-erdat
      FROM vbak
      WHERE vbeln = wa_imdrqx-delnr.  "T_IMDRQX-DELNR.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
  ENDIF.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim
ENDFORM.                    " ACESSA_VBAK

*&---------------------------------------------------------------------*
*&      Form  ACESSA_VBAP
*&---------------------------------------------------------------------*
FORM acessa_vbap .
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
  IF wa_imdrqx-delnr EQ ''.
    SELECT SINGLE kwmeng cuobj untto arktx
  INTO (wa_plaf-kwmeng, wa_plaf-cuobj, wa_plaf-untto, wa_plaf-arktx)
  FROM vbap
  WHERE vbeln = wa_plaf-kdauf
    AND posnr = wa_plaf-kdpos
    AND abgru = ' '.
  ELSE.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim
    SELECT SINGLE kwmeng cuobj untto arktx
      INTO (wa_plaf-kwmeng, wa_plaf-cuobj, wa_plaf-untto, wa_plaf-arktx)
      FROM vbap
      WHERE vbeln = wa_imdrqx-delnr
        AND posnr = wa_imdrqx-delps
        AND abgru = ' '.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
  ENDIF.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim
ENDFORM.                    " ACESSA_VBAP
*&---------------------------------------------------------------------*
*&      Form  ACESSA_VBUP
*&---------------------------------------------------------------------*
FORM acessa_vbup .
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
  IF wa_imdrqx-delnr EQ ''.
    SELECT SINGLE gbsta
  INTO wa_plaf-gbsta
  FROM vbup
  WHERE vbeln =  wa_plaf-kdauf
    AND posnr =  wa_plaf-kdpos
    AND gbsta <> 'C'.
  ELSE.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim
    SELECT SINGLE gbsta
      INTO wa_plaf-gbsta
      FROM vbup
      WHERE vbeln =  wa_imdrqx-delnr
        AND posnr =  wa_imdrqx-delps
        AND gbsta <> 'C'.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
  ENDIF.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim
ENDFORM.                    " ACESSA_VBUP

*&---------------------------------------------------------------------*
*&      Form  ACESSA_VBUK
*&---------------------------------------------------------------------*
FORM acessa_vbuk.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
  IF wa_imdrqx-delnr EQ ''.
    SELECT SINGLE cmgst
      INTO wa_plaf-cmgst
      FROM vbuk
      WHERE vbeln = wa_plaf-kdauf.
  ELSE.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim
    SELECT SINGLE cmgst
    INTO wa_plaf-cmgst
    FROM vbuk
    WHERE vbeln = wa_imdrqx-delnr.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Início
  ENDIF.
****Inserção - Hermiro Carvalho - RDM 1055472 - 08/07/2014 - Fim
ENDFORM.                    " ACESSA_VBUK

*&---------------------------------------------------------------------*
*&      Form  ROTEIRO_MAT_CONFIGURADO
*&---------------------------------------------------------------------*
FORM roteiro_mat_configurado USING p_matnr TYPE mara-matnr
                                   p_werks TYPE mapl-werks.

  CLEAR: v_ferramenta.

  SELECT matnr werks plnty plnnr plnal zkriz zaehl
    FROM mapl
    INTO TABLE t_mapl
    WHERE matnr = p_matnr
      AND werks = p_werks
      AND plnty = 'N'
      AND plnal = '01'
      AND loekz NE 'X'.

  IF sy-subrc = 0.

    SORT t_mapl ASCENDING BY plnty plnnr.
    DELETE ADJACENT DUPLICATES FROM t_mapl COMPARING plnty plnnr.
* INCLUIDO POR SANDRA H. EM 25/04     - INÍCIO

    READ TABLE t_mapl INDEX 1.

    IF sy-subrc = 0.

      SELECT plnnr plnal plnkn zaehl
        INTO TABLE t_plas
        FROM plas
        WHERE plnty = 'N'
          AND plnnr = t_mapl-plnnr
          AND plnal = '01'
          AND loekz NE 'X'.

      IF sy-subrc = 0.
        SELECT plnnr plnkn vornr arbid ltxa1
          INTO TABLE t_plpo
          FROM plpo
          FOR ALL ENTRIES IN t_plas
          WHERE plnty = 'N'
            AND plnnr = t_plas-plnnr
            AND plnkn = t_plas-plnkn
            AND ( steus = 'PP01' OR steus = 'PP02' ).

        IF sy-subrc = 0.

          t_plpo_aux[] = t_plpo[].
          SORT t_plpo_aux ASCENDING BY vornr arbid.
          DELETE ADJACENT DUPLICATES FROM t_plpo_aux COMPARING vornr arbid.

          SELECT objid arbpl ktsch
            INTO TABLE t_crhd_aux
            FROM crhd
            FOR ALL ENTRIES IN t_plpo_aux
            WHERE objid = t_plpo_aux-arbid
              AND verwe <> '0005'
              AND lvorm <> 'X'.

          IF sy-subrc = 0.

*      Ordenar as duas tabelas.
            SORT t_plpo_aux BY vornr.
            SORT t_crhd_aux BY objid.

            LOOP AT t_plpo_aux.

              READ TABLE t_crhd_aux WITH KEY objid = t_plpo_aux-arbid
                                             BINARY SEARCH.
              IF sy-subrc = 0.

                IF v_roteiro IS INITIAL.
                  v_roteiro = t_crhd_aux-arbpl.
                ELSE.
                  CONCATENATE v_roteiro t_crhd_aux-arbpl INTO v_roteiro SEPARATED BY space.
                ENDIF.

                v_maquina = t_crhd_aux-arbpl.
* Busca a ferramenta
                IF t_crhd_aux-ktsch = 'PRESS'.
                  SELECT plnty plnnr objty objid plnkn
                    INTO TABLE t_plfh
                    FROM plfh
                    WHERE plnnr EQ t_plpo_aux-plnnr
                      AND plnkn EQ t_plpo_aux-plnkn
                      AND loekz NE 'X'.
                  IF NOT t_plfh[] IS INITIAL.
                    SORT t_plfh ASCENDING BY objty objid.
                    DELETE ADJACENT DUPLICATES FROM t_plfh COMPARING objty objid.
                    IF t_plfh[] IS NOT INITIAL.
                      LOOP AT t_plfh.
                        SELECT SINGLE *
                          FROM crvm_b
                          WHERE objty EQ t_plfh-objty
                            AND objid EQ t_plfh-objid.
                        IF sy-subrc EQ 0.
                          MOVE crvm_b-matnr TO v_ferramenta.
                        ENDIF.
                      ENDLOOP.
                    ENDIF.

                  ENDIF.

                ENDIF.

                PERFORM de_para_maquina CHANGING v_maquina.

                v_maius = t_plpo_aux-ltxa1.
*           Verifica se são os Roteiros de Produção
                IF sy-tabix    =  1 AND           " ROTEIRO_PRODUCAO1
                   v_maius(12) <> 'MOVIMENTAÇÃO'.
*              Atualiza a tabela principal do Relatório (T_DAD_REL)
                  PERFORM atualiz_dad_rel USING v_maquina.

                ELSEIF sy-tabix    = 1 AND        " ROTEIRO_PRODUCAO1
                       v_maius(12) = 'MOVIMENTAÇÃO'.
*              Atualiza a tabela principal do Relatório (T_DAD_REL)
                  v_centro = '1'.
                  PERFORM atualiz_dad_rel USING v_centro.

                ELSE.
*              Atualiza a tabela principal do Relatório (T_DAD_REL)
                  PERFORM atualiz_dad_rel USING v_maquina.

                ENDIF.
                CLEAR t_crhd_aux.

              ENDIF.

            ENDLOOP.

          ENDIF.
        ENDIF.
      ENDIF.
    ENDIF.
  ENDIF.

ENDFORM.                    " ROTEIRO_MAT_CONFIGURADO

*&---------------------------------------------------------------------*
*&      Form  prazo_entrega_gd
*&---------------------------------------------------------------------*
FORM prazo_entrega_gd .
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*  DATA: v_kdmat TYPE vbap-kdmat VALUE IS INITIAL.
*  SELECT SINGLE kdmat
*    FROM vbap
*    INTO v_kdmat
*    WHERE vbeln = wa_plaf-kdauf AND
*          posnr = wa_plaf-kdpos.

  CLEAR: v_pni, t_tabela_ov[].

  IF wa_plaf-kdauf IS NOT INITIAL.

    CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
      EXPORTING
        input  = ' PDR_PARTNUMBER_INT'
      IMPORTING
        output = v_atinn.

    CALL FUNCTION 'ZRNSDF_SEARCH_COVD_MAE'
      EXPORTING
        ordem_venda              = wa_plaf-kdauf
        item_ordem               = wa_plaf-kdpos
        idioma                   = sy-langu
        caracteristica           = v_atinn
      TABLES
        itab_ov                  = t_tabela_ov
      EXCEPTIONS
        not_found                = 1
        caracteristica_not_found = 2
        ordem_venda_is_missing   = 3
        idioma_is_missing        = 4
        OTHERS                   = 5.
    IF sy-subrc EQ 0.

      READ TABLE t_tabela_ov INDEX 1.
      v_pni = t_tabela_ov-resul.
      IF v_pni = 'X'.
        CLEAR v_pni.
      ENDIF.

    ENDIF.

  ENDIF.

***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim

  v_edatu    = wa_imdrqx-dat00.
* v_edatu = t_imdrqx-dat00.
  IF t_plaf-meins = 'KG'.
    v_wmeng  =  wa_plaf-gsmng / 1000.
*     qte da ordem plan.
  ELSE.
    IF wa_plaf-meins = 'ST' OR wa_plaf-meins = 'PC'.
*    IF wa_plaf-meins = 'ST'.
      PERFORM trata_peca USING wa_plaf-matnr     "#4
                               v_pni.
      v_wmeng = v_qtde * wa_plaf-gsmng.
      wa_plaf-gsmng = v_wmeng.
    ELSE.
      v_wmeng = wa_plaf-gsmng.
    ENDIF.
  ENDIF.

ENDFORM.                    " prazo_entrega_gd

*&---------------------------------------------------------------------*
*&      Form  F_PREENCHE_CLIENTE
*&---------------------------------------------------------------------*
*       Preenche Cliente
*----------------------------------------------------------------------*
FORM f_preenche_cliente .

*Objeto Local
  DATA: vl_strgr TYPE marc-strgr,
        vl_sortl TYPE kna1-sortl,
        vl_kunnr TYPE vbak-kunnr,
        vl_kdmat TYPE vbap-kdmat.

  IF NOT t_dad_rel[] IS INITIAL.

    LOOP AT t_dad_rel.

      CLEAR: vl_sortl, vl_matcli.
      v_tabix = sy-tabix.

*     Verifica estratégia.
      PERFORM f_verifica_estrategia USING t_dad_rel-matnr
                                          t_dad_rel-werks
                                    CHANGING vl_strgr.
* Inclusão - Rodrigo Soeiro - 12/2011 - Inicio
      IF vl_strgr EQ '25'.

        CLEAR vl_kdmat.
        SELECT SINGLE kdmat
          FROM vbap
          INTO vl_kdmat
          WHERE vbeln = t_dad_rel-nr_ov AND
                posnr = t_dad_rel-it_ov.

***Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Início
        CLEAR: vl_kunnr.
        SELECT SINGLE kunnr
          INTO vl_kunnr
          FROM vbak
         WHERE vbeln = t_dad_rel-nr_ov.

        IF vl_kunnr IS INITIAL.
          CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
            EXPORTING
              input  = 'PDR_PARTNUMBER_INT'
            IMPORTING
              output = v_atinn.

          CLEAR t_tabela_ov.
          CALL FUNCTION 'ZRNSDF_SEARCH_COVD_MAE'
            EXPORTING
              ordem_venda              = t_dad_rel-nr_ov
              item_ordem               = t_dad_rel-it_ov
              idioma                   = sy-langu
              caracteristica           = v_atinn
            TABLES
              itab_ov                  = t_tabela_ov
            EXCEPTIONS
              not_found                = 1
              caracteristica_not_found = 2
              ordem_venda_is_missing   = 3
              idioma_is_missing        = 4
              OTHERS                   = 5.
          IF sy-subrc EQ 0.

            READ TABLE t_tabela_ov INDEX 1.
            v_pni = t_tabela_ov-resul.
            IF v_pni = 'X'.
              CLEAR v_pni.
            ENDIF.
          ENDIF.

          SELECT SINGLE kunnr
                  FROM zqmt_partnumber
                  INTO vl_kunnr
                 WHERE partnumber = v_pni AND
                     ( datefrom <= p_period-low  AND dateto >= p_period-high ) AND
                       atnam      = 'SD_MATERIAL_CLIENTE' AND
                     ( status     = 'LIB' OR status = 'LIR' ).
        ENDIF.
****Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Fim
*
****Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Início
*        SORT t_zrnppt_prev BY werks matnr codigo kunnr.
        READ TABLE t_zrnppt_prev WITH KEY werks  = t_dad_rel-werks
                                          matnr  = t_dad_rel-matnr
                                          codigo = vl_kdmat
                                          kunnr  = vl_kunnr.
***Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Fim
        IF sy-subrc EQ 0.
***Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Início
          t_dad_rel-codigo      = t_zrnppt_prev-codigo.
          t_dad_rel-kunnr       = t_zrnppt_prev-kunnr.
***Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Fim
        ELSE.
***Inserção - SANDRA HERNANDEZ - MAIO/2017 - RDM 177618 - INICIO
          READ TABLE t_zrnppt_prev WITH KEY werks  = t_dad_rel-werks2
                                          matnr  = t_dad_rel-matnr
                                          codigo = vl_kdmat
                                          kunnr  = vl_kunnr.
          IF sy-subrc EQ 0.
            t_dad_rel-codigo      = t_zrnppt_prev-codigo.
            t_dad_rel-kunnr       = t_zrnppt_prev-kunnr.

          ENDIF.
***Inserção - SANDRA HERNANDEZ - MAIO/2017 - RDM 177618 - Fim
        ENDIF.
        MODIFY t_dad_rel INDEX v_tabix.
        CLEAR t_dad_rel.
      ENDIF.

      IF vl_strgr EQ '20'.
* Fim da Inclusão
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
        READ TABLE t_zrnppt_prev WITH KEY werks = t_dad_rel-werks
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
                                          matnr = t_dad_rel-matnr.
        IF sy-subrc EQ 0.
          t_dad_rel-codigo = t_zrnppt_prev-codigo.

        ELSE.
***Inserção - SANDRA HERNANDEZ - MAIO/2017 - RDM 177618 - INICIO
          READ TABLE t_zrnppt_prev WITH KEY werks  = t_dad_rel-werks2
                                            matnr  = t_dad_rel-matnr.
          IF sy-subrc EQ 0.
            t_dad_rel-codigo      = t_zrnppt_prev-codigo.
            t_dad_rel-kunnr       = t_zrnppt_prev-kunnr.
          ENDIF .
***Inserção - SANDRA HERNANDEZ - MAIO/2017 - RDM 177618 - INICIO

        ENDIF.

        MODIFY t_dad_rel INDEX v_tabix.
        CLEAR t_dad_rel.
      ENDIF.

      IF vl_strgr EQ '10' OR vl_strgr EQ '70' OR vl_strgr EQ '82'.

*     Verifica se existe na POPA
        PERFORM f_verifica_ordem_producao USING    t_dad_rel-or_pro
                                          CHANGING vl_sortl
                                                   vl_matcli.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
        CLEAR t_zrnppt_prev.
        READ TABLE t_zrnppt_prev WITH KEY werks = t_dad_rel-werks
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
                                          matnr = t_dad_rel-matnr.
*                                          BINARY SEARCH.
        IF t_zrnppt_prev-prod(3) = 'BKS'.
** Busca código do cliente para as Materias-Primas do BKS
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
          READ TABLE t_zrnppt_prev WITH KEY werks = t_dad_rel-werks
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
                                            matnr = t_dad_rel-matnr.
          IF sy-subrc EQ 0.
            t_dad_rel-kunnr  = t_zrnppt_prev-kunnr.
            t_dad_rel-sortl  = t_zrnppt_prev-bismt.
            t_dad_rel-codigo = t_zrnppt_prev-codigo.
          ELSE.
***Inserção - SANDRA HERNANDEZ - MAIO/2017 - RDM 177618 - INICIO
            READ TABLE t_zrnppt_prev WITH KEY werks  = t_dad_rel-werks2
                                              matnr  = t_dad_rel-matnr.
            IF sy-subrc EQ 0.
              t_dad_rel-codigo      = t_zrnppt_prev-codigo.
              t_dad_rel-kunnr       = t_zrnppt_prev-kunnr.
            ENDIF .
***Inserção - SANDRA HERNANDEZ - MAIO/2017 - RDM 177618 - INICIO

          ENDIF.

          MODIFY t_dad_rel INDEX v_tabix.
          CLEAR t_dad_rel.
        ELSE.
***Inserção - Hermiro Carvalho - RDM 105472 - 27/08/2014 - Fim
* Busca código do cliente
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
          READ TABLE t_zrnppt_prev WITH KEY  werks = t_dad_rel-werks
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
                                             matnr = t_dad_rel-matnr.
          IF sy-subrc EQ 0.
            IF t_dad_rel-kunnr IS INITIAL.
              t_dad_rel-kunnr = t_zrnppt_prev-kunnr.
              SELECT SINGLE sortl
                INTO t_dad_rel-sortl
                FROM kna1
                WHERE kunnr = t_dad_rel-kunnr.
            ENDIF.
            IF vl_matcli IS INITIAL.
              t_dad_rel-codigo = t_zrnppt_prev-codigo.
            ENDIF.
          ENDIF.

          IF vl_matcli IS NOT INITIAL.
            t_dad_rel-codigo = vl_matcli.
          ENDIF.
          MODIFY t_dad_rel INDEX v_tabix.
          CLEAR t_dad_rel.
***Inserção - Hermiro Carvalho - RDM 105472 - 27/08/2014 - Início
        ENDIF.
***Inserção - Hermiro Carvalho - RDM 105472 - 27/08/2014 - Fim

      ELSEIF vl_strgr EQ '40'.

* Busca nome código cliente e nome
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
        READ TABLE t_zrnppt_prev WITH KEY werks = t_dad_rel-werks
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
                                          matnr = t_dad_rel-matnr.
        IF sy-subrc EQ 0.
          t_dad_rel-kunnr = t_zrnppt_prev-kunnr.
          t_dad_rel-sortl = t_zrnppt_prev-bismt.
          t_dad_rel-codigo = t_zrnppt_prev-codigo.
        ELSE.
***Inserção - SANDRA HERNANDEZ - MAIO/2017 - RDM 177618 - INICIO
          READ TABLE t_zrnppt_prev WITH KEY werks  = t_dad_rel-werks2
                                            matnr  = t_dad_rel-matnr.
          IF sy-subrc EQ 0.
            t_dad_rel-codigo      = t_zrnppt_prev-codigo.
            t_dad_rel-kunnr       = t_zrnppt_prev-kunnr.
*RDM 190430 - 21.11.17 - Inicio
          ELSE.
            SELECT SINGLE pn_cli kunnr
              INTO (t_dad_rel-codigo, t_dad_rel-kunnr)
              FROM zsdt_part_number
             WHERE material  = t_dad_rel-matnr
             	 AND zf_status = 'A'.
*RDM 190430 - 21.11.17 - Fim
          ENDIF .
***Inserção - SANDRA HERNANDEZ - MAIO/2017 - RDM 177618 - INICIO

        ENDIF.

        MODIFY t_dad_rel INDEX v_tabix.
        CLEAR t_dad_rel.
      ENDIF.
    ENDLOOP.
  ENDIF.

ENDFORM.                    " F_PREENCHE_CLIENTE

*&---------------------------------------------------------------------*
*&      Form  F_VERIFICA_ESTRATEGIA
*&---------------------------------------------------------------------*
*       Verifica Estratégia do Material
*----------------------------------------------------------------------*

FORM f_verifica_estrategia  USING    p_matnr TYPE mara-matnr
                                     p_werks TYPE marc-werks
                            CHANGING p_strgr TYPE marc-strgr.
  CLEAR p_strgr.

  READ TABLE t_marc WITH KEY matnr = p_matnr
                             werks = p_werks
                             BINARY SEARCH.

  IF sy-subrc IS INITIAL.
    p_strgr = t_marc-strgr.
  ELSE.
    SELECT SINGLE strgr
      FROM marc
      INTO p_strgr
      WHERE matnr EQ p_matnr
        AND werks EQ p_werks.
  ENDIF.

ENDFORM.                    " F_VERIFICA_ESTRATEGIA

*&---------------------------------------------------------------------*
*&      Form  F_VERIFICA_ORDEM_PRODUCAO
*&---------------------------------------------------------------------*
*       Verifica Ordem de Produção
*----------------------------------------------------------------------*
FORM f_verifica_ordem_producao  USING    p_aufnr TYPE aufk-aufnr
                                CHANGING p_sortl TYPE kna1-sortl
                                         p_vl_matcli.

  DATA: vl_znumoa  TYPE zrnppt_popa-znumoa,
        vl_znumov  TYPE zrnppt_popa-znumov,
        vl_znomcli TYPE zrnppt_popa-znomcli.

  SELECT znumoa znumov znomcli
    INTO (vl_znumoa, vl_znumov, vl_znomcli)
    FROM   zrnppt_popa
    UP TO 1 ROWS
    WHERE  znumpp = p_aufnr.
  ENDSELECT.

  SELECT SINGLE znomcli zmatcli
    INTO (p_sortl, p_vl_matcli)
    FROM zrnppt_pres
    WHERE znumoa EQ vl_znumoa
      AND znumpp EQ p_aufnr.

  CONDENSE p_vl_matcli NO-GAPS. "Leandro Projeto Unir2

ENDFORM.                    " F_VERIFICA_ORDEM_PRODUCAO

*&---------------------------------------------------------------------*
*&      Form  F_SELECIONA_ZRNPPT_PREV
*&---------------------------------------------------------------------*
FORM f_seleciona_zrnppt_prev .

  DATA: vl_espes(8)      TYPE c,
        vl_larg(8)       TYPE c,
        vl_compr(8)      TYPE c,
        vl_base_maior(8) TYPE c,
        vl_base_menor(8) TYPE c,
        vl_cod_prot      TYPE ze_cod_prot,
        vl_tabix         LIKE sy-tabix.

***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Início
  DATA: v_mes  	       TYPE zrnppt_prev-mes,
        v_ano          TYPE zrnppt_prev-ano,
        wa_zrnppt_prev TYPE zrnppt_prev.
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Fim

  DATA: v_kzkfg         TYPE mara-kzkfg.

***Inicio - Fabio - 24.03.2021 - RDM 256631
  DATA: BEGIN OF t_peca_peso OCCURS 0,
          peca TYPE zrnppt_prev-peca,
          peso TYPE zrnppt_prev-peso,
        END OF t_peca_peso.
***Fim - Fabio - 24.03.2021 - RDM 256631


  CLEAR: t_zrnppt_prev, t_zrnppt_prev[], t_melhoria[].

  IF p_mat[] IS INITIAL AND p_matpri[] IS NOT INITIAL.

    "Configurado
    IF t_mast[] IS NOT INITIAL.
      CLEAR: wa_mast.
      LOOP AT t_mast INTO wa_mast.
        p_mat-sign   = 'I'.
        p_mat-option = 'EQ'.
        p_mat-low    = wa_mast-matnr.
        APPEND p_mat.
        CLEAR: wa_mast.
      ENDLOOP.
    ENDIF.

    "Configurável ou Configurado com PNI
    IF t_partn_mrp[] IS NOT INITIAL.
      LOOP AT t_partn_mrp WHERE matnr IS NOT INITIAL.
        CLEAR p_mat.
        p_mat-sign = 'I'.
        p_mat-option = 'EQ'.
        p_mat-low = t_partn_mrp-matnr.
        APPEND p_mat.
      ENDLOOP.
    ENDIF.

    IF p_matpri[] IS NOT INITIAL.

      "Material IN Natura
      SELECT matnr
        INTO TABLE t_matnr
        FROM mara
       WHERE matnr IN p_matpri.
      IF sy-subrc IS INITIAL.
        LOOP AT t_matnr.
          CLEAR p_mat.
          p_mat-sign = 'I'.
          p_mat-option = 'EQ'.
          p_mat-low = t_matnr-matnr.
          APPEND p_mat.
        ENDLOOP.
        CLEAR: t_matnr[].
      ENDIF.

    ENDIF.

    IF p_mat[] IS NOT INITIAL.
      SORT p_mat ASCENDING BY low.
      DELETE ADJACENT DUPLICATES FROM p_mat COMPARING low.
    ENDIF.

  ENDIF.

  IF p_mat[] IS INITIAL AND p_matpri[] IS INITIAL AND r_tudo IS INITIAL.
    PERFORM f_ajusta_p_mat.
  ENDIF.

  IF NOT p_codigo[] IS INITIAL.
*     seleciona tabela de previsões de produção
    SELECT werks bismt ano mes matnr espes
           larg compr base_maior base_menor
           codigo prod pn entr peso_peca embalagem op1 op2 op3 op4 op5 op6 op7 op8
           peso data_modif ferramenta kunnr tamanho_lote estoque_minimo estoque_min_pc peca
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
           ativo status_pcp status_om
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
     FROM zrnppt_prev
     INTO TABLE t_zrnppt_prev
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
*     WHERE werks = p_centro-low AND
     WHERE werks IN p_centro AND
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
*Alteração - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
*           ano =  v_ano_de   AND
*           mes IN r_mes      AND
*Alteração - Sidney Q. de Andrade - SU - 03.01.2012 - Fim
*Inclusão - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
             ano IN r_ano    AND
*Inclusão - Sidney Q. de Andrade - SU - 03.01.2012 - Fim
           matnr IN p_mat    AND
           kunnr IN p_clt    AND
*           codigo = p_codigo AND
           codigo IN p_codigo AND
           ( op1 IN s_cent_t OR
             op2 IN s_cent_t OR
             op3 IN s_cent_t OR
             op4 IN s_cent_t OR
             op5 IN s_cent_t OR
             op6 IN s_cent_t OR
             op7 IN s_cent_t OR
             op8 IN s_cent_t ).
  ELSE.
* seleciona tabela de previsões de produção
    SELECT werks bismt ano mes matnr espes
           larg compr base_maior base_menor
           codigo prod pn entr peso_peca embalagem op1 op2 op3 op4 op5 op6 op7 op8
           peso data_modif ferramenta kunnr tamanho_lote estoque_minimo estoque_min_pc peca
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
           ativo status_pcp status_om
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
     FROM zrnppt_prev
     INTO TABLE t_zrnppt_prev
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
*     WHERE werks = p_centro-low AND
     WHERE werks IN p_centro AND
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
*Alteração - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
*           ano =  v_ano_de   AND
*           mes IN r_mes      AND
*Alteração - Sidney Q. de Andrade - SU - 03.01.2012 - Fim
*Inclusão - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
             ano IN r_ano    AND
*Inclusão - Sidney Q. de Andrade - SU - 03.01.2012 - Fim
           matnr IN p_mat    AND
           kunnr IN p_clt    AND
           ( op1 IN s_cent_t OR
             op2 IN s_cent_t OR
             op3 IN s_cent_t OR
             op4 IN s_cent_t OR
             op5 IN s_cent_t OR
             op6 IN s_cent_t OR
             op7 IN s_cent_t OR
             op8 IN s_cent_t ).
  ENDIF.

  IF sy-subrc = 0.

*RDM322490 - GVCA20241122 - Begin
    IF stdem1 = 'X'.
      DELETE t_zrnppt_prev WHERE NOT ( ativo = 'S' AND peso = '0' ).
    ELSEIF stdem2 = 'X'.
      DELETE t_zrnppt_prev WHERE NOT ( ativo = 'S' AND peso > '0' ).
    ENDIF.

    CHECK t_zrnppt_prev[] IS NOT INITIAL.
*RDM322490 - GVCA20241122 - End

    PERFORM f_seleciona_marc.

    "Filtrar por periodo, componente, partnumber interno e status
    LOOP AT t_zrnppt_prev.

      vl_tabix = sy-tabix.


      "Check EXTRANET PRODEXT
      IF p_gd IS NOT INITIAL.
        CLEAR t_marc.
        READ TABLE t_marc WITH KEY matnr = t_zrnppt_prev-matnr
                                   werks = t_zrnppt_prev-werks
                                   BINARY SEARCH.
        IF t_marc-sobsl <> c_30.
          DELETE t_zrnppt_prev INDEX vl_tabix.
        ENDIF.
      ENDIF.



*RDM 203086
      "Check Deposito
      IF s_lgort[] IS NOT INITIAL.
        CLEAR t_marc.
        READ TABLE t_marc WITH KEY matnr = t_zrnppt_prev-matnr
                                   werks = t_zrnppt_prev-werks
                                   BINARY SEARCH.
        IF t_marc-lgpro NOT IN s_lgort[].
          DELETE t_zrnppt_prev INDEX vl_tabix.
        ENDIF.
      ENDIF.
*RDM 203086

      "Check por Periodo
      CONCATENATE t_zrnppt_prev-ano t_zrnppt_prev-mes '01' INTO v_periodo.
*      IF ( v_periodo < p_period-low ) OR ( v_periodo > p_period-high ).
      IF ( v_periodo(6) < p_period-low(6)  ) OR ( v_periodo(6) > p_period-high(6) ).  "RDM 321837-correção seleção período
        DELETE t_zrnppt_prev INDEX vl_tabix.
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Início
        CONTINUE.
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Fim
      ENDIF.
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
***Inserção - Hermiro Carvalho - RDM 138179 - 04/05/2015 - Início
*      IF p_ativo IS NOT INITIAL."Se a opção Ativo estiver selecionada
*        IF t_zrnppt_prev-peso IS INITIAL."Verifica só o que tem valor no mês no período definido no parâmetro
*          DELETE t_zrnppt_prev INDEX sy-tabix.
****Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Início
*          CONTINUE.
****Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Fim
*        ENDIF."Verifica só o que tem valor no mês no período definido no parâmetro
*      ENDIF."Se a opção Ativo estiver selecionada
***Inserção - Hermiro Carvalho - RDM 138179 - 04/05/2015 - Fim
****Caso a opção p_todos estiver selecionada
*   If p_todos is NOT INITIAL.
*
*   endif.

      "Check por Componente - exclusivo para o cenário: Material Configuravel com Componente cadastrado no Partnumber
      IF p_matpri[] IS NOT INITIAL.
        CLEAR v_kzkfg.
        SELECT SINGLE kzkfg
        FROM mara
        INTO v_kzkfg
          WHERE matnr = t_zrnppt_prev-matnr.
        IF v_kzkfg IS NOT INITIAL.
          READ TABLE t_partn_mrp WITH KEY matnr = t_zrnppt_prev-matnr
                                          kunnr = t_zrnppt_prev-kunnr
                                          kdmat = t_zrnppt_prev-codigo.
          IF sy-subrc IS NOT INITIAL.
            DELETE t_zrnppt_prev INDEX vl_tabix.
            CONTINUE.
          ENDIF.
        ENDIF.
      ENDIF.

***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
      "Check por Partnumber Interno
      IF p_pni[] IS NOT INITIAL.
        CLEAR v_pni.
        SELECT SINGLE partnumber
          FROM zqmt_partnumber
          INTO v_pni
          WHERE matnr = t_zrnppt_prev-matnr AND
                kunnr = t_zrnppt_prev-kunnr AND
                kdmat = t_zrnppt_prev-codigo AND
             ( datefrom <= p_period-low  AND dateto >= p_period-high ) AND
             ( status    = 'LIB' OR status = 'LIR' ).
        IF v_pni NOT IN p_pni.
          DELETE t_zrnppt_prev INDEX vl_tabix.
          CONTINUE.
        ENDIF.
      ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim

      "Check por Status
      IF p_ativo IS NOT INITIAL.
        IF t_zrnppt_prev-ativo <> 'S'."Verifica se é ativo.
          DELETE t_zrnppt_prev INDEX vl_tabix.
          CONTINUE.
        ENDIF.
      ENDIF.
***Inserção - Hermiro Carvalho - RDM 140328 - 21/07/2015 - Início
      IF p_inati IS NOT INITIAL.
        IF t_zrnppt_prev-ativo <> 'N'."Verifica se é ativo.
          DELETE t_zrnppt_prev INDEX vl_tabix.
          CONTINUE.
        ENDIF.
      ENDIF.
***Inserção - Hermiro Carvalho - RDM 140328 - 21/07/2015 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim
    ENDLOOP.

    CHECK t_zrnppt_prev[] IS NOT INITIAL.

* Grazielle Alves - Procwork - Início - 17/09/2008 - Projeto Complementar - PP128

    IF NOT p_fer IS INITIAL.
      SORT t_zrnppt_prev BY ferramenta.
      DELETE t_zrnppt_prev WHERE NOT ferramenta IN p_fer.
    ENDIF.
* Grazielle Alves - Procwork - Fim    - 17/09/2008 - Projeto Complementar - PP128

    t_zrnppt_prev_aux[] = t_zrnppt_prev[].
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*    SORT t_zrnppt_prev BY werks matnr codigo.
*    SORT t_zrnppt_prev_aux BY werks matnr codigo.
    SORT t_zrnppt_prev BY werks matnr codigo kunnr ASCENDING data_modif DESCENDING.
    SORT t_zrnppt_prev_aux BY werks matnr codigo kunnr ASCENDING data_modif DESCENDING.

*    delete adjacent duplicates from t_zrnppt_prev comparing werks matnr codigo.
    DELETE ADJACENT DUPLICATES FROM t_zrnppt_prev COMPARING werks matnr codigo kunnr.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
    SELECT matnr bismt meins spart kzkfg
      INTO TABLE t_mara
      FROM mara
      FOR ALL ENTRIES IN t_zrnppt_prev
      WHERE matnr = t_zrnppt_prev-matnr.

    IF t_mara[] IS NOT INITIAL.
      SORT t_mara BY matnr.

      SELECT matnr meinh umrez umren
        INTO TABLE t_marm
        FROM marm
        FOR ALL ENTRIES IN t_mara
        WHERE matnr = t_mara-matnr.

      SORT t_marm BY matnr meinh.
    ENDIF.

*RDM 291540
    SELECT *
      INTO TABLE @DATA(lt_mast)
      FROM mast
      FOR ALL ENTRIES IN @t_zrnppt_prev
      WHERE matnr EQ @t_zrnppt_prev-matnr
        AND werks EQ @t_zrnppt_prev-werks
        AND stlan EQ '1'.
    IF sy-subrc IS INITIAL.
      SELECT *
        INTO TABLE @DATA(lt_stpo)
        FROM stpo
        FOR ALL ENTRIES IN @lt_mast
        WHERE stlty EQ 'M'
          AND stlnr EQ @lt_mast-stlnr
          AND lkenz EQ ''.
    ENDIF.
    SORT lt_mast BY matnr werks.
    SORT lt_stpo BY stlnr.
*RDM 291540

    LOOP AT t_zrnppt_prev.
      v_tabix = sy-tabix.
      CLEAR: t_zrnppt_prev-peso, t_zrnppt_prev-peca.

      READ TABLE t_mara WITH KEY matnr = t_zrnppt_prev-matnr.

      IF r_jit = 'X'.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*        IF t_mara-kzkfg <> 'JT'.
        IF t_mara-spart <> 'JT'.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
          DELETE t_zrnppt_prev[] INDEX v_tabix.
          CONTINUE.
        ENDIF.
      ENDIF.

      IF r_dist = 'X'.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*        IF t_mara-kzkfg <> 'JT'.
        IF t_mara-spart = 'JT'.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
          DELETE t_zrnppt_prev[] INDEX v_tabix.
          CONTINUE.
        ENDIF.
      ENDIF.


      LOOP AT t_zrnppt_prev_aux WHERE matnr  = t_zrnppt_prev-matnr  AND
                                      codigo = t_zrnppt_prev-codigo AND
                                      werks  = t_zrnppt_prev-werks  AND
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
                                      kunnr = t_zrnppt_prev-kunnr.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início

        ADD t_zrnppt_prev_aux-peso TO t_zrnppt_prev-peso.
        ADD t_zrnppt_prev_aux-peca TO t_zrnppt_prev-peca.

        IF t_zrnppt_prev-ferramenta IS INITIAL.
          t_zrnppt_prev-ferramenta = t_zrnppt_prev_aux-ferramenta.
        ENDIF.

      ENDLOOP.

      MODIFY t_zrnppt_prev INDEX v_tabix.
***Inserção - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Início
      t_saida-werks          = t_zrnppt_prev-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Fim
      t_saida-cliente        = t_zrnppt_prev-bismt.
      t_saida-matnr          = t_zrnppt_prev-matnr.
      t_saida-codigo         = t_zrnppt_prev-codigo.              "Partnumber do Cliente / SD_MATERIAL_CLIENTE
      t_saida-kdmat          = t_zrnppt_prev-codigo.              "Partnumber do Cliente / SD_MATERIAL_CLIENTE
***Inserção - Hermiro Carvalho - RDM 138179 - 04/05/2015 - Início
      IF t_zrnppt_prev-peso IS INITIAL.
        t_saida-prev_org       = 0.
      ELSE.
        t_saida-prev_org       = t_zrnppt_prev-peso.
      ENDIF.
      t_saida-ativo            = t_zrnppt_prev-ativo.
***Inserção - Hermiro Carvalho - RDM 138179 - 04/05/2015 - Fim
      t_saida-ferramenta     = t_zrnppt_prev-ferramenta.
      t_saida-cod_cli        = t_zrnppt_prev-kunnr.
      t_saida-tamanho_lote   = t_zrnppt_prev-tamanho_lote.
***Inserção - Hermiro Carvalho - PRJ01313 - 15/09/2015 - Início
      t_saida-pn             = t_zrnppt_prev-pn.                    "Núm.do desenho /ou Nro.da Peça
*      t_saida-pn             = t_zrnppt_prev-codigo.
***Inserção - Hermiro Carvalho - PRJ01313 - 22/09/2015 - Início
      SELECT SINGLE partnumber datefrom dateto
        FROM zqmt_partnumber
        INTO (t_saida-pn_interno, t_saida-datefrom, t_saida-dateto)
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*       WHERE kdmat = t_saida-pn AND
        WHERE matnr = t_saida-matnr   AND
              kunnr = t_saida-cod_cli AND
              kdmat = t_saida-codigo AND
           ( datefrom <= p_period-low  AND dateto >= p_period-high ) AND
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
             atnam      = 'SD_MATERIAL_CLIENTE' AND
           ( status     = 'LIB' OR status = 'LIR' ).
***Inserção - Hermiro Carvalho - PRJ01313 - 22/09/2015 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Início
***Comentar - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Início
*      CLEAR: v_vbeln, v_posnr, v_kdmat_aux.
*      SELECT SINGLE a~vbeln a~posnr a~kdmat
*        INTO (v_vbeln, v_posnr, v_kdmat_aux)
*        FROM vbap AS a INNER JOIN vbak AS b
*          ON ( a~vbeln = b~vbeln )
*       WHERE a~matnr = t_saida-matnr AND
*             a~kdmat = t_saida-pn    AND
*             a~werks = t_saida-werks AND
*             b~auart = '01' AND
****Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*             b~kunnr = t_saida-cod_cli.
      CLEAR: v_vbeln, v_posnr.
      SELECT SINGLE vbeln
        INTO v_vbeln
        FROM zqmt_partn_mrp
       WHERE partnumber = t_saida-pn_interno
         AND werks = t_saida-werks.
      IF sy-subrc IS INITIAL.
        SELECT SINGLE vbeln posnr
          INTO (v_vbeln, v_posnr)
          FROM vbap
         WHERE vbeln = v_vbeln.
      ENDIF.
***Comentar - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
      CONCATENATE v_vbeln v_posnr INTO t_saida-ov_md81 SEPARATED BY '/'.
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 15/09/2015 - Fim
      t_saida-estoque_minimo = t_zrnppt_prev-estoque_minimo.
      t_saida-embalagem      = t_zrnppt_prev-embalagem.
* Bruno Quadros - CSC - 30.09.2010 - Inicio
      t_saida-peso_peca      = t_zrnppt_prev-peso_peca.
* Bruno Quadros - CSC - 30.09.2010 - Fim

* Bruno Quadros - CSC - 12.04.2011 - Inicio
      IF t_zrnppt_prev-peso_peca IS NOT INITIAL.
        t_saida-est_min_pec  = ( t_zrnppt_prev-estoque_minimo * 1000 ) / t_zrnppt_prev-peso_peca.
      ENDIF.
      t_saida-prev_org_pec   = t_zrnppt_prev-peca.
* Bruno Quadros - CSC - 12.04.2011 - Fim

*     Define estratégia
      READ TABLE t_marc WITH KEY matnr = t_zrnppt_prev-matnr
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Início
                                 werks = t_zrnppt_prev-werks
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Fim
                                 BINARY SEARCH.
      IF sy-subrc = 0.
        t_saida-strgr = t_marc-strgr.
*RDM 203086
        t_saida-lgort = t_marc-lgpro.
*RDM 203086
      ENDIF.

      CONDENSE: t_zrnppt_prev-espes NO-GAPS,
                t_zrnppt_prev-larg NO-GAPS,
                t_zrnppt_prev-compr NO-GAPS,
                t_zrnppt_prev-base_maior NO-GAPS,
                t_zrnppt_prev-base_menor NO-GAPS.

      vl_espes      = t_zrnppt_prev-espes.
      vl_larg       = t_zrnppt_prev-larg.
      vl_compr      = t_zrnppt_prev-compr.
      vl_base_maior = t_zrnppt_prev-base_maior.
      vl_base_menor = t_zrnppt_prev-base_menor.

      IF t_saida-strgr = '25'.
        IF t_zrnppt_prev-compr IS INITIAL.

          CONCATENATE t_zrnppt_prev-prod '-' vl_espes 'x' vl_larg  'x' vl_base_maior 'x'
                      vl_base_menor INTO t_saida-descricao SEPARATED BY space.
        ELSE.
          CONCATENATE t_zrnppt_prev-prod '-' vl_espes 'x' vl_larg  'x'
                      vl_compr INTO t_saida-descricao SEPARATED BY space.
        ENDIF.
      ELSE.
        PERFORM f_texto_material USING    t_zrnppt_prev-matnr
                                 CHANGING t_saida-descricao.
      ENDIF.

*     Atribui as operações
      CONCATENATE t_zrnppt_prev-op1 t_zrnppt_prev-op2 t_zrnppt_prev-op3
                  t_zrnppt_prev-op4 t_zrnppt_prev-op5 t_zrnppt_prev-op6
                  t_zrnppt_prev-op7 t_zrnppt_prev-op8 INTO t_saida-roteiro
                  SEPARATED BY space.
***Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Início
**Seleciona dados na kna1
      SELECT SINGLE kunnr sortl
        FROM kna1
        INTO wa_kna1
       WHERE kunnr = t_zrnppt_prev-entr.

      t_saida-data_modif      = t_zrnppt_prev-data_modif.
      READ TABLE t_mara WITH KEY matnr = t_zrnppt_prev-matnr
                                 BINARY SEARCH.
      t_saida-meins           = t_mara-meins.
      t_saida-werks           = t_zrnppt_prev-werks.
      t_saida-h_material      = t_zrnppt_prev-prod.
      t_saida-l_entrega       = wa_kna1-sortl.

***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
      SELECT SINGLE matnr werks kunnr stock_min
        FROM zsuppt_estoq_dia
        INTO wa_zsuppt_estoq_dia
       WHERE matnr = t_zrnppt_prev-matnr
         AND werks = t_zrnppt_prev-werks
         AND kunnr = t_zrnppt_prev-kunnr.

      IF wa_zsuppt_estoq_dia IS INITIAL.
        SELECT SINGLE matnr werks kunnr stock_min
          FROM zsuppt_estoq_dia
          INTO wa_zsuppt_estoq_dia
         WHERE matnr = t_zrnppt_prev-matnr
           AND werks = t_zrnppt_prev-werks.

      ENDIF.

      CLEAR: vl_cod_prot.
      IF wa_zsuppt_estoq_dia IS INITIAL.

        SELECT SINGLE cod_prot
          FROM zsdt_otif_pnblen
          INTO vl_cod_prot
          WHERE pn_cli   = t_zrnppt_prev-codigo AND
                lentrega = t_zrnppt_prev-kunnr.
        IF sy-subrc <> 0.
          SELECT SINGLE cod_prot
            FROM knvv
            INTO vl_cod_prot
            WHERE kunnr =  t_zrnppt_prev-kunnr AND
                  spart = 'JT'.
        ENDIF.

        IF vl_cod_prot IS NOT INITIAL.
          SELECT SINGLE qtdestmin
            FROM zsdt_otif_protoc
            INTO wa_zsuppt_estoq_dia-stock_min
            WHERE cod_prot = vl_cod_prot.
        ENDIF.

      ENDIF.

** Sandra H. RDM  174627
      CLEAR wa_marc.
      SELECT SINGLE matnr werks rwpro
        INTO wa_marc
        FROM marc
        WHERE matnr = t_saida-matnr
          AND werks = t_saida-werks.

** INICIO: RGC8096 30jan2017 RDM 175234
*      SELECT SINGLE rwpro mirwx
*        INTO wa_t438u
*        FROM t438u
*       WHERE rwpro EQ wa_marc-rwpro.
*      IF wa_t438u-mirwx > 0.
*        t_saida-est_min_dias = wa_t438u-mirwx.
*      ENDIF.
** FIM: RGC8096 30jan2017 RDM 175234
** Sandra Hernandez RDM 174627

* Início da Alteração - Nelson Junior - Stefanini - 18.08.2025
** INICIO: RGC8096 30jan2017 RDM 175234
*      CLEAR wa_t438u.
*      SELECT SINGLE rwpro rw1tg
*        INTO wa_t438u
*        FROM t438r
****Inserção - Hermiro Carvalho - INC 2384066 - 13/07/2017 - Início
*       WHERE werks EQ wa_marc-werks
*         AND rwpro EQ wa_marc-rwpro.
****Inserção - Hermiro Carvalho - INC 2384066 - 13/07/2017 - Fim
*      IF sy-subrc IS INITIAL.
*        IF wa_t438u-mirwx > 0.
*          t_saida-est_min_dias = wa_t438u-mirwx.
*        ENDIF.
*      ENDIF.
** FIM: RGC8096 30jan2017 RDM 175234
* Início da Alteração - Nelson Junior - Stefanini - 18.08.2025

*    t_saida-est_min_dias     = wa_zsuppt_estoq_dia-stock_min.
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Início
***Preenche o campo periodo +1 - Início
      CLEAR: v_mes, wa_zrnppt_prev, v_ano.
      v_mes = t_zrnppt_prev-mes + 1.
      v_ano = t_zrnppt_prev-ano.
      IF v_mes > 12.
        v_mes = 01.
        v_ano = t_zrnppt_prev-ano + 1.
      ENDIF.

      SELECT SINGLE peca peso
        INTO (wa_zrnppt_prev-peca, wa_zrnppt_prev-peso)
        FROM zrnppt_prev
       WHERE werks = t_zrnppt_prev-werks
***Inserção - Hermiro Carvalho - PRJ01313 - 03/11/2015 - Início
*         AND bismt = t_zrnppt_prev-bismt
***Inserção - Hermiro Carvalho - PRJ01313 - 03/11/2015 - Fim
         AND ano   = v_ano
         AND mes   = v_mes
         AND matnr = t_zrnppt_prev-matnr
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
         AND codigo = t_zrnppt_prev-codigo
         AND kunnr = t_zrnppt_prev-kunnr.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
*         AND espes = t_zrnppt_prev-espes
*         AND larg  = t_zrnppt_prev-larg
*         AND compr = t_zrnppt_prev-compr.
      IF sy-subrc IS INITIAL.
        t_saida-periodo_1pc = wa_zrnppt_prev-peca.
        t_saida-periodo_1 = wa_zrnppt_prev-peso.
      ENDIF.
*    ENDIF.
***Preenche o campo periodo +1 - Fim

***Preenche o campo periodo +2 - Início
      CLEAR: wa_zrnppt_prev.
      v_mes = v_mes + 1.
*      v_ano = t_zrnppt_prev-ano.
      IF v_mes > 12.
        v_mes = 01.
        v_ano = t_zrnppt_prev-ano + 1.
      ENDIF.
      SELECT SINGLE peca peso
        INTO (wa_zrnppt_prev-peca, wa_zrnppt_prev-peso)
        FROM zrnppt_prev
       WHERE werks = t_zrnppt_prev-werks
***Inserção - Hermiro Carvalho - PRJ01313 - 03/11/2015 - Início
*         AND bismt = t_zrnppt_prev-bismt
***Inserção - Hermiro Carvalho - PRJ01313 - 03/11/2015 - Fim
         AND ano   = v_ano
         AND mes   = v_mes
         AND matnr = t_zrnppt_prev-matnr
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
         AND codigo = t_zrnppt_prev-codigo
         AND kunnr = t_zrnppt_prev-kunnr.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
*         AND espes = t_zrnppt_prev-espes
*         AND larg  = t_zrnppt_prev-larg
*         AND compr = t_zrnppt_prev-compr.
      IF sy-subrc IS INITIAL.
        t_saida-periodo_2pc = wa_zrnppt_prev-peca.
        t_saida-periodo_2 = wa_zrnppt_prev-peso.
      ENDIF.
***Preenche o campo periodo +2 - Fim

***Inserção - Sidney Q de Andrade - RDM 164690 - 27/06/2016 - Inicio
***Preenche o campo periodo +3 - Início
      CLEAR: wa_zrnppt_prev.
      v_mes = v_mes + 1.
      IF v_mes > 12.
        v_mes = 01.
        v_ano = t_zrnppt_prev-ano + 1.
      ENDIF.
      SELECT SINGLE peca peso
        INTO (wa_zrnppt_prev-peca, wa_zrnppt_prev-peso)
        FROM zrnppt_prev
       WHERE werks = t_zrnppt_prev-werks
         AND ano   = v_ano
         AND mes   = v_mes
         AND matnr = t_zrnppt_prev-matnr
         AND codigo = t_zrnppt_prev-codigo
         AND kunnr = t_zrnppt_prev-kunnr.
      IF sy-subrc IS INITIAL.
        t_saida-periodo_3pc = wa_zrnppt_prev-peca.
        t_saida-periodo_3 = wa_zrnppt_prev-peso.
      ENDIF.
***Preenche o campo periodo +3 - Fim

***Preenche o campo periodo +4 - Início
      CLEAR: wa_zrnppt_prev.
      v_mes = v_mes + 1.
      IF v_mes > 12.
        v_mes = 01.
        v_ano = t_zrnppt_prev-ano + 1.
      ENDIF.
      SELECT SINGLE peca peso
        INTO (wa_zrnppt_prev-peca, wa_zrnppt_prev-peso)
        FROM zrnppt_prev
       WHERE werks = t_zrnppt_prev-werks
         AND ano   = v_ano
         AND mes   = v_mes
         AND matnr = t_zrnppt_prev-matnr
         AND codigo = t_zrnppt_prev-codigo
         AND kunnr = t_zrnppt_prev-kunnr.
      IF sy-subrc IS INITIAL.
        t_saida-periodo_4pc = wa_zrnppt_prev-peca.
        t_saida-periodo_4 = wa_zrnppt_prev-peso.
      ENDIF.
***Preenche o campo periodo +4 - Fim

***Preenche o campo periodo +5 - Início
      CLEAR: wa_zrnppt_prev.
      v_mes = v_mes + 1.
      IF v_mes > 12.
        v_mes = 01.
        v_ano = t_zrnppt_prev-ano + 1.
      ENDIF.
      SELECT SINGLE peca peso
        INTO (wa_zrnppt_prev-peca, wa_zrnppt_prev-peso)
        FROM zrnppt_prev
       WHERE werks = t_zrnppt_prev-werks
         AND ano   = v_ano
         AND mes   = v_mes
         AND matnr = t_zrnppt_prev-matnr
         AND codigo = t_zrnppt_prev-codigo
         AND kunnr = t_zrnppt_prev-kunnr.
      IF sy-subrc IS INITIAL.
        t_saida-periodo_5pc = wa_zrnppt_prev-peca.
        t_saida-periodo_5 = wa_zrnppt_prev-peso.
      ENDIF.

***Inicio - Fabio - 24.03.2021 - RDM 256631
      CLEAR: t_peca_peso[].

      SELECT peca peso
        INTO TABLE t_peca_peso
        FROM zrnppt_prev
       WHERE werks  <> t_zrnppt_prev-werks
         AND ano    = t_zrnppt_prev-ano
         AND mes    = t_zrnppt_prev-mes
         AND matnr  = t_zrnppt_prev-matnr
         AND codigo = t_zrnppt_prev-codigo
         AND kunnr  = t_zrnppt_prev-kunnr.

      IF sy-subrc IS INITIAL.
        CLEAR: t_saida-prev_org_cen,
               t_saida-prev_org_cen_pec.

        LOOP AT t_peca_peso.
          t_saida-prev_org_cen     = t_saida-prev_org_cen + t_peca_peso-peso.
          t_saida-prev_org_cen_pec = t_saida-prev_org_cen_pec + t_peca_peso-peca.
        ENDLOOP.
      ENDIF.
***Fim - Fabio - 24.03.2021 - RDM 256631

***Preenche o campo periodo +5 - Fim
***Inserção - Sidney Q de Andrade - RDM 164690 - 27/06/2016 - Fim

***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Fim
*      t_saida-est_min_pc       = t_zrnppt_prev-estoque_min_pc.
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
*      APPEND t_saida.
***Inativo(p_inati marcado com X) vai aqui
****Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*      IF p_inati IS NOT INITIAL."Se a opção Inativo estiver selecionada
*        IF t_zrnppt_prev-ativo = 'N'."Verifica só o que tem valor período definido no parâmetro
*          DELETE t_zrnppt_prev INDEX sy-tabix.
*        ENDIF."Verifica só o que tem valor período definido no parâmetro
*      ENDIF."Se a opção Inativo estiver selecionada
***Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Início
      SELECT SINGLE sobsl
        FROM marc
        INTO v_sobsl
        WHERE matnr EQ t_saida-matnr
          AND werks EQ t_saida-werks.

      IF v_sobsl = c_30.
        t_saida-producao = 'EXTERNA'.
      ELSE.
        t_saida-producao = 'INTERNA'.
      ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Fim
****Inserção - Hermiro Carvalho - RDM 160097 - 04/05/2016 - Início
*      CLEAR: wa_melhoria.
*      wa_melhoria-werks      = t_zrnppt_prev-werks.
*      wa_melhoria-matnr      = t_zrnppt_prev-matnr.
*      APPEND wa_melhoria TO t_melhoria.
****Inserção - Hermiro Carvalho - RDM 160097 - 04/05/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 168377 - 08/09/2016 - Início
      SELECT SINGLE projeto
        INTO t_saida-projeto
        FROM zsdt_part_number
       WHERE material  = t_zrnppt_prev-matnr
         AND kunnr     = t_zrnppt_prev-kunnr
       	 AND zf_status = 'A'.
***Inserção - Hermiro Carvalho - RDM 168377 - 08/09/2016 - Fim


      t_saida-status_pcp =  t_zrnppt_prev-status_pcp.
      t_saida-status_om  =  t_zrnppt_prev-status_om.
***Inicio - Fabio - 25.01.2021 - RDM 251520
      SELECT SINGLE klabc
                  FROM knvv
                  INTO t_saida-tp_cliente
                  WHERE kunnr =  t_zrnppt_prev-kunnr AND
                        spart = 'JT'.
***Fim - Fabio - 25.01.2021 - RDM 251520

      APPEND t_saida.

* Bruno Quadros - CSC - 30.09.2010 - Inicio
      IF t_saida-descricao(3) = 'BKS' OR
*RDM 240520 - INICIO
         t_saida-descricao(3) = 'PLT'.

*        DATA: v_aux_kdmat TYPE char35 VALUE IS INITIAL.
*        v_aux_kdmat = t_saida-codigo.
*RDM 240520 - FIM

        PERFORM trata_peca USING t_saida-matnr          "#5
                                 t_saida-pn_interno.
*INC 3097794
*        PERFORM f_append_mat_prima.
        PERFORM f_append_mat_intermediario.
*INC 3097794
*RDM 291540
      ELSE.
        "verifica se lista técnica contém material intermediário
        READ TABLE lt_mast INTO DATA(ls_mast) WITH KEY matnr = t_saida-matnr
                                                       werks = t_saida-werks
                                                       BINARY SEARCH.
        IF sy-subrc IS INITIAL.
          READ TABLE lt_stpo INTO DATA(ls_stpo)
                             WITH KEY stlnr = ls_mast-stlnr
                             TRANSPORTING NO FIELDS
                             BINARY SEARCH.
          IF sy-subrc IS INITIAL.
            LOOP AT lt_stpo INTO ls_stpo FROM sy-tabix.
              IF ls_stpo-stlnr NE ls_mast-stlnr.
                EXIT.
              ELSE.
                IF ls_stpo-idnrk(2) EQ '54'.
*INC 3770039
                  PERFORM trata_peca USING t_saida-matnr
                                           t_saida-pn_interno.
*INC 3770039
                  PERFORM f_append_mat_intermediario.
                  EXIT.
                ENDIF.
              ENDIF.
            ENDLOOP.
          ENDIF.
        ENDIF.
*RDM 291540
      ENDIF.
* Bruno Quadros - CSC - 30.09.2010 - Fim
****Inserção - Hermiro Carvalho - RDM 105472 - 22/07/2014 - Início
***Seleciona dados na kna1
*      SELECT SINGLE kunnr sortl
*        FROM kna1
*        INTO wa_kna1
*       WHERE kunnr = t_zrnppt_prev-entr.
*
*      t_saida-data_modif      = t_zrnppt_prev-data_modif.
*      READ TABLE t_mara WITH KEY matnr = t_zrnppt_prev-matnr
*                                 BINARY SEARCH.
*      t_saida-meins           = t_mara-meins.
*      t_saida-werks           = t_zrnppt_prev-werks.
*      t_saida-h_material      = t_zrnppt_prev-prod.
*      t_saida-l_entrega       = wa_kna1-sortl.
*
****Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
*      SELECT SINGLE matnr werks kunnr stock_min
*        FROM zsuppt_estoq_dia
*        INTO wa_zsuppt_estoq_dia
*       WHERE matnr = t_zrnppt_prev-matnr
*         AND werks = t_zrnppt_prev-werks
*         AND kunnr = t_zrnppt_prev-kunnr.
*
*      IF wa_zsuppt_estoq_dia IS INITIAL.
*        SELECT SINGLE matnr werks kunnr stock_min
*          FROM zsuppt_estoq_dia
*          INTO wa_zsuppt_estoq_dia
*         WHERE matnr = t_zrnppt_prev-matnr
*           AND werks = t_zrnppt_prev-werks.
*
*      ENDIF.
*
*      t_saida-est_min_dias     = wa_zsuppt_estoq_dia-stock_min.
**      t_saida-est_min_pc       = t_zrnppt_prev-estoque_min_pc.
****Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
**      APPEND t_saida.
****Inativo(p_inati marcado com X) vai aqui
*****Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
**      IF p_inati IS NOT INITIAL."Se a opção Inativo estiver selecionada
**        IF t_zrnppt_prev-ativo = 'N'."Verifica só o que tem valor período definido no parâmetro
**          DELETE t_zrnppt_prev INDEX sy-tabix.
**        ENDIF."Verifica só o que tem valor período definido no parâmetro
**      ENDIF."Se a opção Inativo estiver selecionada
****Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Início
*      MODIFY t_saida INDEX v_tabix.
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Fim

***Inserção - Hermiro Carvalho - RDM 105472 - 22/07/2014 - Fim
      CLEAR: t_saida, t_zrnppt_prev.




    ENDLOOP.
* Bruno Quadros - CSC - 30.09.2010 - Início
    IF t_zrnppt_prev_aux2[] IS NOT INITIAL.
      APPEND LINES OF t_zrnppt_prev_aux2 TO t_zrnppt_prev.

      SELECT matnr bismt meins
        APPENDING TABLE t_mara
        FROM mara
        FOR ALL ENTRIES IN t_zrnppt_prev_aux2
        WHERE matnr = t_zrnppt_prev_aux2-matnr.

      IF t_mara[] IS NOT INITIAL.
        SORT t_mara BY matnr.

        SELECT matnr meinh umrez umren
          APPENDING TABLE t_marm
          FROM marm
          FOR ALL ENTRIES IN t_zrnppt_prev_aux2
          WHERE matnr = t_zrnppt_prev_aux2-matnr.

        SORT t_marm BY matnr meinh.
      ENDIF.

    ENDIF.
* Bruno Quadros - CSC - 30.09.2010 - Fim

  ENDIF.

* Comentado e transferido para para o perform centro_produtor
***Inserção - Hermiro Carvalho - RDM 160097 - 04/05/2016 - Início
*  IF t_saida[] IS NOT INITIAL.
*
*    LOOP AT t_saida.
*      CLEAR: wa_melhoria.
*      wa_melhoria-werks      = t_saida-werks.
*      wa_melhoria-matnr      = t_saida-matnr.
*      wa_melhoria-wrk02      = t_saida-wrk02. " 09mar2017 - RGC8096 - RDM 176853
*      APPEND wa_melhoria TO t_melhoria.
*    ENDLOOP.
*
*    SORT t_melhoria BY werks matnr.
*    DELETE ADJACENT DUPLICATES FROM t_melhoria COMPARING werks matnr.
*
*  ENDIF.
***Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Fim

  PERFORM f_carrega_ovs_emitidas.

* INICIO: RGC8096 RDM 191755 24Jan2018
*  LOOP AT t_saida.
*    v_tabix = sy-tabix.
*    READ TABLE t_saida_prev WITH KEY matnr = t_saida-matnr
*                                     werks = t_saida-werks.
*    IF sy-subrc EQ 0.
*      t_saida-ov_emitida = t_saida_prev-programa.
*      MODIFY t_saida INDEX v_tabix.
*    ENDIF.
*  ENDLOOP.
* FIM   : RGC8096 RDM 191755  24Jan2018

ENDFORM.                    " F_SELECIONA_ZRNPPT_PREV

*&---------------------------------------------------------------------*
*&      Form  F_TEXTO_MATERIAL
*&---------------------------------------------------------------------*
FORM f_texto_material  USING    p_matnr TYPE mara-matnr
                       CHANGING p_descricao.

  SELECT SINGLE maktx
    FROM makt
    INTO p_descricao
    WHERE matnr = p_matnr
      AND spras = sy-langu.

ENDFORM.                    " F_TEXTO_MATERIAL

*&---------------------------------------------------------------------*
*&      Form  f_carrega_ov_op
*&---------------------------------------------------------------------*

FORM f_carrega_ov_op .

  DATA: v_kdmat   TYPE vbap-kdmat,
        v_partaux TYPE zqmt_partn_mrp-partnumber.
***Comentar - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Início
*  SELECT SINGLE kdmat
*    FROM vbap
*    INTO v_kdmat
*    WHERE vbeln = wa_plaf_aux-kdauf AND
*          posnr = wa_plaf_aux-kdpos.
***Comentar - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Fim

  DATA: vl_tabix LIKE sy-tabix.

  SORT t_dad_rel BY werks kunnr matnr codigo.

*RDM 190430 - 21.11.17 – Inicio
  CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
    EXPORTING
      input  = 'SD_PESO_BLANK'
    IMPORTING
      output = v_atinn_pesopeca.
*RDM 190430 - 21.11.17 – Fim


  LOOP AT t_saida.
    vl_tabix = sy-tabix.
    CLEAR: t_saida-ov, t_saida-op.

    READ TABLE t_dad_rel WITH KEY werks  = t_saida-werks
                                  kunnr  = t_saida-cod_cli
                                  matnr  = t_saida-matnr
                                  codigo = t_saida-codigo
                                  BINARY SEARCH.

    IF sy-subrc = 0.
      LOOP AT t_dad_rel FROM sy-tabix.
        v_tabix = sy-tabix.

        IF t_dad_rel-werks  <> t_saida-werks   OR
           t_dad_rel-kunnr  <> t_saida-cod_cli OR
           t_dad_rel-matnr  <> t_saida-matnr   OR
           t_dad_rel-codigo <> t_saida-codigo.
*** Inserção - Sandra H - abril/2017 - RDM 177618 - Início
          IF t_saida-werks <> t_saida-wrk02.  " se FOR PRODUÇÃO EM OUTRO CENTRO

            IF t_dad_rel-werks  <> t_saida-wrk02   OR
               t_dad_rel-matnr  <> t_saida-matnr.
              EXIT.
            ELSE.
              PERFORM prod_outro_centro.
            ENDIF.
          ELSE.
            EXIT.
          ENDIF.
*** Inserção - Sandra H - abril/2017 - RDM 177618 - FIM
        ELSE.
          t_saida-ov = t_saida-ov + t_dad_rel-qte_pl.
          t_saida-op = t_saida-op + t_dad_rel-qte_or.
          IF t_saida-ferramenta IS INITIAL.
            t_saida-ferramenta = t_dad_rel-ferramenta.
          ENDIF.
* Verifica o status da OP
          SELECT SINGLE idat2 ftrmi
            INTO (caufv-idat2, caufv-ftrmi)
            FROM caufv
            WHERE aufnr EQ t_dad_rel-or_pro.
          IF sy-subrc EQ 0.
            IF caufv-idat2 = '00000000'.

              CLEAR t_zrnppt_prev.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
              READ TABLE t_zrnppt_prev WITH KEY  werks = t_dad_rel-werks
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
                                                 kunnr = t_saida-cod_cli
                                                 matnr = t_dad_rel-matnr
                                                 codigo = t_dad_rel-codigo.

              IF caufv-ftrmi = '00000000'.
                t_saida-op_aber = t_saida-op_aber + t_dad_rel-qte_or.
                IF t_zrnppt_prev-peso_peca IS NOT INITIAL.
                  t_saida-op_aber_pec = ( t_saida-op_aber * 1000 )  / t_zrnppt_prev-peso_peca.
                  t_dad_rel-op_peca   = ( t_dad_rel-qte_or * 1000 ) / t_zrnppt_prev-peso_peca.
                ENDIF.
                t_dad_rel-op_aber = t_dad_rel-qte_or.
              ELSE.
                t_saida-op_lib = t_saida-op_lib + t_dad_rel-qte_or.
                IF t_zrnppt_prev-peso_peca IS NOT INITIAL.
                  t_saida-op_lib_pec = ( t_saida-op_lib * 1000 )   / t_zrnppt_prev-peso_peca.
                  t_dad_rel-op_peca  = ( t_dad_rel-qte_or * 1000 ) / t_zrnppt_prev-peso_peca.
                ENDIF.
                t_dad_rel-op_lib = t_dad_rel-qte_or.
              ENDIF.

* Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
*              SELECT znumoa znumov zitemov UP TO 1 ROWS
*                INTO (t_dad_rel-op, t_dad_rel-nr_ov, t_dad_rel-it_ov)
*                FROM zrnppt_popa
*                WHERE znumpp = t_dad_rel-or_pro.
*              ENDSELECT.

              READ TABLE t_aufk  WITH KEY aufnr = t_dad_rel-or_pro.
              IF sy-subrc IS INITIAL.
                SELECT znumoa znumov zitemov UP TO 1 ROWS
                  INTO (t_dad_rel-op, t_dad_rel-nr_ov, t_dad_rel-it_ov)
                  FROM zrnppt_popa
                  WHERE znumoa = t_aufk-cy_seqnr.
                ENDSELECT.
              ENDIF.
* Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Fim
* Início da Alteração - Nelson Junior - Stefanini - 18.08.2025
*             PERFORM: zf_qualifica_demanda.
* Fim da Alteração - Nelson Junior - Stefanini - 18.08.2025
              MODIFY t_dad_rel INDEX v_tabix.
            ENDIF.
          ENDIF.
        ENDIF.
      ENDLOOP.
    ENDIF.

*RDM 190430  - 17.11.17 - Inicio
    "Totalizar OPs nos Outros Centros que não possui demanda
    LOOP AT t_dad_rel WHERE werks <> t_saida-werks   AND
                            kunnr  = t_saida-cod_cli AND
                            matnr  = t_saida-matnr   AND
                            codigo = t_saida-codigo.

      v_tabix = sy-tabix.

*INC 3283615
*      "não considerar OPs de centro com demanda
*      CLEAR t_zrnppt_prev.
*      READ TABLE t_zrnppt_prev WITH KEY  werks  = t_dad_rel-werks
*                                         kunnr  = t_dad_rel-kunnr
*                                         matnr  = t_dad_rel-matnr
*                                         codigo = t_dad_rel-codigo.
*      IF sy-subrc IS INITIAL.
*        CONTINUE.
*      ENDIF.
*INC 3283615

      SELECT SINGLE idat2 ftrmi
        INTO (caufv-idat2, caufv-ftrmi)
        FROM caufv
        WHERE aufnr EQ t_dad_rel-or_pro.
      IF sy-subrc EQ 0.

        IF caufv-idat2 = '00000000'.

          CLEAR: t_caractmat[], t_caractmat.
          CALL FUNCTION 'ZRNMMF_SEARCH_CMAT'
            EXPORTING
              matnr                     = t_dad_rel-matnr
              klart                     = '001'
              idioma                    = sy-langu
              caracteristica            = v_atinn_pesopeca
            TABLES
              itab_mat                  = t_caractmat
            EXCEPTIONS
              matnr_not_found           = 1
              matnr_or_mara_not_found   = 2
              caracteristicas_not_found = 3
              classe_not_found          = 4
              OTHERS                    = 5.
          READ TABLE t_caractmat INDEX 1.
          t_zrnppt_prev-peso_peca = t_caractmat-atflv.

          IF caufv-ftrmi = '00000000'.
            t_saida-op_aber_cen = t_saida-op_aber_cen + t_dad_rel-qte_or.
            IF t_zrnppt_prev-peso_peca IS NOT INITIAL.
              t_saida-op_aber_cen_pec = ( t_saida-op_aber_cen * 1000 ) / t_zrnppt_prev-peso_peca.
              t_dad_rel-op_peca   = ( t_dad_rel-qte_or * 1000 ) / t_zrnppt_prev-peso_peca.
            ENDIF.
            t_dad_rel-op_aber = t_dad_rel-qte_or.
          ELSE.
            t_saida-op_lib_cen = t_saida-op_lib_cen + t_dad_rel-qte_or.
            IF t_zrnppt_prev-peso_peca IS NOT INITIAL.
              t_saida-op_lib_cen_pec = ( t_saida-op_lib_cen * 1000 ) / t_zrnppt_prev-peso_peca.
              t_dad_rel-op_peca  = ( t_dad_rel-qte_or * 1000 ) / t_zrnppt_prev-peso_peca.
            ENDIF.
            t_dad_rel-op_lib = t_dad_rel-qte_or.
          ENDIF.

          READ TABLE t_aufk  WITH KEY aufnr = t_dad_rel-or_pro.
          IF sy-subrc IS INITIAL.
            SELECT znumoa znumov zitemov UP TO 1 ROWS
              INTO (t_dad_rel-op, t_dad_rel-nr_ov, t_dad_rel-it_ov)
              FROM zrnppt_popa
              WHERE znumoa = t_aufk-cy_seqnr.
            ENDSELECT.
          ENDIF.
          MODIFY t_dad_rel INDEX v_tabix.

        ENDIF.

      ENDIF.

    ENDLOOP.
*RDM 190430  - 17.11.17 - Fim

    IF t_saida-ov IS NOT INITIAL AND t_saida-peso_peca IS NOT INITIAL.
      t_saida-ov_peca = ( t_saida-ov * 1000 ) / t_saida-peso_peca.
    ENDIF.
    MODIFY t_saida INDEX vl_tabix.
  ENDLOOP.

***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Início
***Select na PLAF para preencher os campos planejadas, data inicio
  CLEAR: t_plaf_aux[].
  SELECT plnum matnr plwrk gsmng psttr meins kdauf kdpos
    INTO TABLE t_plaf_aux
    FROM plaf
     FOR ALL ENTRIES IN t_saida
   WHERE matnr EQ t_saida-matnr
     AND plwrk EQ t_saida-werks.

  IF t_plaf_aux[] IS NOT INITIAL.
    SORT: t_plaf_aux BY plnum matnr plwrk psttr.
  ENDIF.

  CLEAR: vl_tabix.
  LOOP AT t_saida.

    vl_tabix = sy-tabix.
*** Sandra Hernandez - abril/2017 - RDM 177618 - Início
*    READ TABLE t_marc WITH KEY matnr = t_saida-matnr
*                               werks = t_saida-werks.

*   LOOP AT t_plaf_aux INTO wa_plaf_aux WHERE matnr = t_marc-matnr
*                                          AND plwrk = t_marc-werks.
    LOOP AT t_plaf_aux INTO wa_plaf_aux WHERE matnr = t_saida-matnr
                                          AND plwrk = t_saida-werks.
*** Sandra Hernandez - abril/2017 - RDM 177618 - Fim
      CLEAR: v_material, v_kdmat, v_kunnr.

      IF wa_plaf_aux-kdauf IS NOT INITIAL.

        SELECT SINGLE kdmat
          FROM vbap
          INTO v_kdmat
          WHERE vbeln = wa_plaf_aux-kdauf AND
                posnr = wa_plaf_aux-kdpos.

***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Início
        CLEAR: v_pni, t_tabela_ov[].

        CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
          EXPORTING
            input  = 'PDR_PARTNUMBER_INT'
          IMPORTING
            output = v_atinn.

        CALL FUNCTION 'ZRNSDF_SEARCH_COVD_MAE'
          EXPORTING
            ordem_venda              = wa_plaf_aux-kdauf
            item_ordem               = wa_plaf_aux-kdpos
            idioma                   = sy-langu
            caracteristica           = v_atinn
          TABLES
            itab_ov                  = t_tabela_ov
          EXCEPTIONS
            not_found                = 1
            caracteristica_not_found = 2
            ordem_venda_is_missing   = 3
            idioma_is_missing        = 4
            OTHERS                   = 5.
        IF sy-subrc EQ 0.
          READ TABLE t_tabela_ov INDEX 1.
          v_pni = t_tabela_ov-resul.
          IF v_pni = 'X'.
            CLEAR v_pni.
          ENDIF.
        ENDIF.

        CLEAR: v_partaux.
        SELECT SINGLE kunnr
          INTO v_kunnr
          FROM vbak
         WHERE vbeln = wa_plaf_aux-kdauf.
        IF v_kunnr IS INITIAL.
          SELECT SINGLE partnumber
            INTO v_partaux
            FROM zqmt_partn_mrp
           WHERE vbeln = wa_plaf_aux-kdauf.

          SELECT SINGLE kunnr
            INTO v_kunnr
            FROM zqmt_partnumber
           WHERE partnumber = v_partaux.
        ENDIF.

      ELSE.

        READ TABLE t_zrnppt_prev WITH KEY werks = wa_plaf_aux-plwrk
                                          matnr = wa_plaf_aux-matnr.
        IF sy-subrc EQ 0.
          v_kdmat = t_zrnppt_prev-codigo.
          v_kunnr = t_zrnppt_prev-kunnr.
        ENDIF.

      ENDIF.

      v_material = wa_plaf_aux-matnr.

      IF  v_kunnr     <> t_saida-cod_cli OR
          v_material  <> t_saida-matnr   OR
          v_kdmat     <> t_saida-codigo.
        CONTINUE.
      ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Fim

***Inserção - Hermiro Carvalho - RDM 138179 - 11/05/2015 - Início
      IF wa_plaf_aux-meins = 'KG'.
        wa_plaf_aux-gsmng  =  wa_plaf_aux-gsmng / 1000.
      ELSEIF wa_plaf_aux-meins = 'ST' OR wa_plaf_aux-meins = 'PC'.
        PERFORM trata_peca USING t_saida-matnr          "#6
                                 t_saida-pn_interno.
        wa_plaf_aux-gsmng =  v_qtde * wa_plaf_aux-gsmng.
      ENDIF.
***Inserção - Hermiro Carvalho - RDM 138179 - 11/05/2015 - Fim
***Calcula o campo data_inicio e planejadas.
      IF t_saida-data_inicio IS INITIAL OR wa_plaf_aux-psttr < t_saida-data_inicio.
        t_saida-data_inicio = wa_plaf_aux-psttr.
***Inserção - Hermiro Carvalho - RDM 185165 - 09/08/2017 - Início
        CLEAR t_marc-shzet.
        SELECT SINGLE shzet
          INTO t_marc-shzet
          FROM marc
         WHERE matnr = t_saida-matnr
           AND werks = t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 185165 - 09/08/2017 - Fim
        CASE t_marc-shzet.
          WHEN 00.
            t_saida-data_inicio = t_saida-data_inicio - 5.

            CALL FUNCTION 'DATE_CONVERT_TO_FACTORYDATE' " Conversão de data para dia útil.
              EXPORTING
                correct_option               = '-'
                date                         = t_saida-data_inicio
                factory_calendar_id          = 'BR'
              IMPORTING
                date                         = t_saida-data_inicio
*               factorydate                  =
*               workingday_indicator         =
              EXCEPTIONS
                calendar_buffer_not_loadable = 1
                correct_option_invalid       = 2
                date_after_range             = 3
                date_before_range            = 4
                date_invalid                 = 5
                factory_calendar_not_found   = 6
                OTHERS                       = 7.

            CLEAR: t_saida-planejadas.
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Início
*              t_saida-planejadas  = wa_plaf_aux-gsmng.
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Início
*                  t_saida-planejadas_pc  = wa_plaf_aux-gsmng.
            t_saida-planejadas     = wa_plaf_aux-gsmng.
            IF t_saida-planejadas IS NOT INITIAL AND t_saida-peso_peca IS NOT INITIAL.
              t_saida-planejadas_pc =  t_saida-planejadas / ( t_saida-peso_peca / 1000 ).
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Fim
*              t_saida-planejadas     = wa_plaf_aux-gsmng.
          WHEN OTHERS.
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Início
*                  t_saida-planejadas_pc  = wa_plaf_aux-gsmng.
            t_saida-planejadas     = wa_plaf_aux-gsmng.
            IF t_saida-planejadas IS NOT INITIAL AND t_saida-peso_peca IS NOT INITIAL.
              t_saida-planejadas_pc =  t_saida-planejadas / ( t_saida-peso_peca / 1000 ).
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Fim
*              t_saida-planejadas     = wa_plaf_aux-gsmng.
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Fim
        ENDCASE.
*          MODIFY t_saida INDEX vl_tabix.
      ELSE.
        CONTINUE.
      ENDIF.
    ENDLOOP.

*** Sandra Hernandez - abril/2017 - RDM 177618 - Início
*    LOOP AT t_aufk WHERE matnr = t_marc-matnr
*                     AND werks = t_marc-werks.
    LOOP AT t_aufk WHERE matnr = t_saida-matnr
                     AND werks = t_saida-wrk02.
*** Sandra Hernandez - abril/2017 - RDM 177618 - Fim

      CLEAR: v_material, v_kdmat, v_kunnr.

      IF t_aufk-kdauf IS NOT INITIAL.
        SELECT SINGLE kdmat
          FROM vbap
          INTO v_kdmat
          WHERE vbeln = t_aufk-kdauf AND
                posnr = t_aufk-kdpos.

****Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Início
        CLEAR: t_tabela_ov[], v_pni.
        CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
          EXPORTING
            input  = 'PDR_PARTNUMBER_INT'
          IMPORTING
            output = v_atinn.

        CALL FUNCTION 'ZRNSDF_SEARCH_COVD_MAE'
          EXPORTING
            ordem_venda              = t_aufk-kdauf
            item_ordem               = t_aufk-kdpos
            idioma                   = sy-langu
            caracteristica           = v_atinn
          TABLES
            itab_ov                  = t_tabela_ov
          EXCEPTIONS
            not_found                = 1
            caracteristica_not_found = 2
            ordem_venda_is_missing   = 3
            idioma_is_missing        = 4
            OTHERS                   = 5.
        IF sy-subrc EQ 0.
          READ TABLE t_tabela_ov INDEX 1.
          v_pni = t_tabela_ov-resul.
          IF v_pni = 'X'.
            CLEAR v_pni.
          ENDIF.
        ENDIF.

        CLEAR: v_partaux.
        SELECT SINGLE kunnr
          INTO v_kunnr
          FROM vbak
         WHERE vbeln = t_aufk-kdauf.
        IF v_kunnr IS INITIAL.
          SELECT SINGLE partnumber
            INTO v_partaux
            FROM zqmt_partn_mrp
           WHERE vbeln = t_aufk-kdauf.

          SELECT SINGLE kunnr
            INTO v_kunnr
            FROM zqmt_partnumber
           WHERE partnumber = v_partaux.
        ENDIF.

      ELSE.

*** Alteração Sandra H. abri/2017 - RDM 177618 - INICIO
*        READ TABLE t_zrnppt_prev WITH KEY werks = t_aufk-werks
*                                          matnr = t_aufk-matnr.

        READ TABLE t_zrnppt_prev WITH KEY werks = t_saida-werks
                                          matnr = t_aufk-matnr.

*** Alteração Sandra H. abri/2017 - RDM 177618 - FIM
        IF sy-subrc EQ 0.
          v_kdmat = t_zrnppt_prev-codigo.
          v_kunnr = t_zrnppt_prev-kunnr.
        ENDIF.

      ENDIF.

      v_material = t_aufk-matnr.

      IF  v_kunnr     <> t_saida-cod_cli OR
          v_material  <> t_saida-matnr  OR
          v_kdmat     <> t_saida-codigo.
        CONTINUE.
      ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Fim

* Verifica o status da OP
      SELECT SINGLE idat2 ftrmi
        INTO (caufv-idat2, caufv-ftrmi)
        FROM caufv
        WHERE aufnr EQ t_aufk-aufnr.
      IF sy-subrc EQ 0.
        IF caufv-ftrmi = '00000000'.
          IF t_saida-produzir_op IS INITIAL OR t_aufk-gstrs < t_saida-prazo.
            CLEAR: t_saida-produzir_op, t_saida-prazo.
            t_saida-produzir_op = t_aufk-gamng.
            t_saida-prazo       = t_aufk-gstrs.
          ENDIF.
        ENDIF.
      ENDIF.

    ENDLOOP.

    MODIFY t_saida INDEX vl_tabix.

  ENDLOOP.

*  ENDIF.
***Inserção - Hermiro Carvalho - RDM 137207 - 13/04/2015 - Fim

ENDFORM.                    " f_carrega_ov_op

*&---------------------------------------------------------------------*
*&      Form  F_EXPEDICAO
*&---------------------------------------------------------------------*
FORM f_expedicao .

* Verifica saldo na MSKA
  PERFORM f_saldo_mska.

* Verifica saldo na MCHB
  PERFORM f_saldo_mchb.

ENDFORM.                    " F_EXPEDICAO

*&---------------------------------------------------------------------*
*&      Form  F_SALDO_MSKA
*&---------------------------------------------------------------------*
FORM f_saldo_mska .

  DATA: t_vbap TYPE TABLE OF vbap WITH HEADER LINE.

  DATA: vl_partnumber  TYPE zqmt_partnumber-partnumber.

  SELECT werks lgort matnr charg kalab ersda kains vbeln posnr
    FROM mska
    INTO TABLE t_mska
    FOR ALL ENTRIES IN t_zrnppt_prev
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*    WHERE werks = p_centro-low
     WHERE werks = t_zrnppt_prev-werks
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
      AND matnr = t_zrnppt_prev-matnr
      AND ( kalab GT 0 OR kains GT 0 ).

  IF t_mska[] IS NOT INITIAL.
    SELECT vbeln posnr kdmat
      FROM vbap
      INTO CORRESPONDING FIELDS OF TABLE t_vbap
      FOR ALL ENTRIES IN t_mska
      WHERE vbeln = t_mska-vbeln
        AND posnr = t_mska-posnr.
  ENDIF.

  IF t_mska[] IS NOT INITIAL.

    SELECT vbeln kunnr auart
      FROM vbak
      INTO TABLE t_vbak
      FOR ALL ENTRIES IN t_mska
      WHERE vbeln = t_mska-vbeln.

**********************************************************************
* Obter os clientes das OVs de Estoque do tipo 5000xxxx
**********************************************************************
    LOOP AT t_vbak WHERE auart = '01'.

      v_tabix = sy-tabix.

      CLEAR vl_partnumber.
      SELECT SINGLE partnumber
        FROM zqmt_partn_mrp
        INTO vl_partnumber
        WHERE vbeln = t_vbak-vbeln.

      SELECT SINGLE kunnr
        FROM zqmt_partnumber
        INTO t_vbak-kunnr
       WHERE partnumber = vl_partnumber AND
             atnam      = 'SD_MATERIAL_CLIENTE' AND
           ( status     = 'LIB' OR status = 'LIR' ).

      MODIFY t_vbak INDEX v_tabix.

    ENDLOOP.
**********************************************************************

    IF t_vbak[] IS NOT INITIAL.

      SELECT kunnr sortl
        FROM kna1
        INTO TABLE t_kna1
        FOR ALL ENTRIES IN t_vbak
        WHERE kunnr = t_vbak-kunnr.

      IF sy-subrc = 0.

        LOOP AT t_mska.

          v_tabix = sy-tabix.

          READ TABLE t_vbak WITH KEY vbeln = t_mska-vbeln.
          IF sy-subrc = 0.
            READ TABLE t_kna1 WITH KEY kunnr = t_vbak-kunnr.
            IF sy-subrc = 0.
              t_mska-sortl = t_kna1-sortl.
              t_mska-kunnr = t_kna1-kunnr.
              MODIFY t_mska INDEX v_tabix.
            ENDIF.
          ENDIF.

          READ TABLE t_vbap WITH KEY vbeln = t_mska-vbeln
                                     posnr = t_mska-posnr.
          t_mska-kdmat = t_vbap-kdmat.
          MODIFY t_mska INDEX v_tabix.

        ENDLOOP.

      ENDIF.

    ENDIF.

    SELECT matnr werks strgr minbe bstma shzet lgpro sobsl
      APPENDING TABLE t_marc
      FROM marc
      FOR ALL ENTRIES IN t_mska
      WHERE matnr = t_mska-matnr
***Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*        AND werks = p_centro-low.
        AND werks = t_mska-werks.
***Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim

    SORT t_marc BY matnr werks.
    DELETE ADJACENT DUPLICATES FROM t_marc COMPARING ALL FIELDS.

***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Início
*    LOOP AT t_mska.
*
*      v_tabix = sy-tabix.
*      CLEAR t_marc.
*      READ TABLE t_marc WITH KEY matnr = t_mska-matnr
****Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*                                 werks = t_mska-werks.
****Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
*      IF sy-subrc = 0.
*        CASE t_marc-strgr.
*
*          WHEN '40'
** Inclusão - Rodrigo Soeiro - 12/2011 - Inicio
*            OR '20'.
** Fim da inclusão
*            CLEAR: v_ok,
*                   v_ferramenta.
*            PERFORM check_mat_configurado  USING t_mska-matnr
*                                                 t_mska-werks
*                                             CHANGING v_ok.
*            t_mska-ferramenta = v_ferramenta.
*
*            MODIFY t_mska INDEX v_tabix.
*
*          WHEN '25'.
*            CLEAR: v_ok, vl_matcli.
*            v_klart = '300'.
*            PERFORM check_mat_configuravel USING v_klart
*                                                t_mska-vbeln
*                                                t_mska-posnr
*                                            CHANGING v_ok
*                                                     vl_matcli.
*            t_mska-ferramenta = v_ferramenta.
*            t_mska-matcli     = vl_matcli.
*
*            MODIFY t_mska INDEX v_tabix.
*
*          WHEN '80'.
*            CLEAR: v_ok,
*                   v_ferramenta.
*            PERFORM check_mat_configurado  USING t_mska-matnr
*                                                 t_mska-werks
*                                             CHANGING v_ok.
*            t_mska-ferramenta = v_ferramenta.
*
*            MODIFY t_mska INDEX v_tabix.
*
*          WHEN '10' OR '82'.
*            CLEAR: v_ok,
*                   v_ferramenta.
*            PERFORM check_mat_configurado  USING t_mska-matnr
*                                                 t_mska-werks
*                                             CHANGING v_ok.
*            t_mska-ferramenta = v_ferramenta.
*
*            MODIFY t_mska INDEX v_tabix.
*
*        ENDCASE.
*      ENDIF.
*    ENDLOOP.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim

* Atribui expedição a tabela de saida
*    SORT t_mska BY matnr matcli ferramenta.
    SORT t_mska BY werks matnr kunnr kdmat.

    LOOP AT t_saida.

      v_tabix = sy-tabix.

      CLEAR: v_expedi, v_mat_cq.

***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
*      LOOP AT t_mska WHERE matnr      = t_saida-matnr AND
      LOOP AT t_mska WHERE werks      = t_saida-werks AND
                           matnr      = t_saida-matnr AND
                           kunnr      = t_saida-cod_cli AND
                           kdmat      = t_saida-codigo.
*                           ferramenta = t_saida-ferramenta.
*                           matcli     = t_saida-codigo AND
*                           ferramenta = t_saida-ferramenta.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim

        IF t_mska-kalab IS NOT INITIAL AND t_mska-lgort <> '0096'."*****Hermiro 09/02/2017
          v_expedi = v_expedi + t_mska-kalab.
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Início
*        ELSEIF t_mska-kains IS NOT INITIAL.
        ELSEIF t_mska-kains IS NOT INITIAL OR t_mska-lgort = '0096'.
*            v_expedicao = v_expedicao + t_mska-kains.
*          v_mat_cq = v_mat_cq + t_mska-kains.
          v_mat_cq = v_mat_cq + t_mska-kains +  t_mska-kalab.
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Fim
        ENDIF.

      ENDLOOP.

      t_saida-expedicao = v_expedi.
      t_saida-mat_cq    = v_mat_cq.
      t_saida-analise_prev  = t_saida-expedicao - t_saida-estoque_minimo.
***Inserção - Hermiro Carvalho - PRJ01313 - 15/09/2015 - Início
      IF t_saida-estoque_minimo > 0.
        t_saida-analise       = ( t_saida-analise_prev / t_saida-estoque_minimo ).
      ELSE.
        t_saida-analise       = 0.
      ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 15/09/2015 - Fim
      MODIFY t_saida INDEX v_tabix.
    ENDLOOP.

  ENDIF.

ENDFORM.                    " F_SALDO_MSKA

*&---------------------------------------------------------------------*
*&      Form  check_MAT_CONFIGURADO
*&---------------------------------------------------------------------*
FORM check_mat_configurado  USING     p_matnr TYPE mara-matnr
                                      p_werks TYPE mapl-werks
                             CHANGING p_ok.

  DATA: vl_tabix LIKE sy-tabix.
  DATA: p_beskz  TYPE c.
  CLEAR: v_ferramenta.

  SELECT matnr werks plnty plnnr plnal zkriz zaehl
    FROM mapl
    INTO TABLE t_mapl
    WHERE matnr = p_matnr
      AND werks = p_werks
      AND plnty = 'N'
*      AND PLNAL = '01'
      AND loekz NE 'X'.

  IF sy-subrc = 0.

    SORT t_mapl ASCENDING BY plnty plnnr.
    DELETE ADJACENT DUPLICATES FROM t_mapl COMPARING plnty plnnr.

    SELECT plnnr plnkn vornr arbid ltxa1
      INTO TABLE t_plpo
      FROM plpo
      FOR ALL ENTRIES IN t_mapl
      WHERE plnty = t_mapl-plnty
        AND plnnr = t_mapl-plnnr
        AND ( steus = 'PP01' OR steus = 'PP02' ).

    IF sy-subrc = 0.
      SORT t_crhd BY objid.

      LOOP AT t_plpo.
        vl_tabix = sy-tabix.
        READ TABLE t_crhd WITH KEY objid = t_plpo-arbid.
*                                  BINARY SEARCH.
        IF sy-subrc <> 0.
          DELETE t_plpo INDEX vl_tabix.
        ENDIF.
      ENDLOOP.
      IF NOT t_plpo[]  IS INITIAL.
        p_ok = 'X'.
        t_plpo_aux[] = t_plpo[].
        SORT t_plpo_aux ASCENDING BY arbid.
        DELETE ADJACENT DUPLICATES FROM t_plpo_aux COMPARING arbid.

        SELECT objid arbpl FROM crhd
          INTO TABLE t_crhd_aux
          FOR ALL ENTRIES IN t_plpo_aux
* Projeto Unir2 - Fabiana - 28.08.2009 - Início
*          WHERE objid = t_plpo_aux-arbid.
          WHERE objid = t_plpo_aux-arbid
            AND verwe <> '0005'.
* Projeto Unir2 - Fabiana - 28.08.2009 - Fim

        IF sy-subrc = 0.

*      Ordenar as duas tabelas.
          SORT t_plpo_aux BY vornr.
          SORT t_crhd_aux BY objid.

          LOOP AT t_plpo_aux.

            READ TABLE t_crhd_aux WITH KEY objid = t_plpo_aux-arbid
                                           BINARY SEARCH.
            IF sy-subrc = 0.
*            V_FLAG = '1'.       "Indica que possui centro de trabalho
              v_maquina = t_crhd_aux-arbpl.
* Busca a ferramenta
              IF v_maquina(1) = 'P'.
                SELECT plnty plnnr objty objid plnkn
                  FROM plfh
                  INTO TABLE t_plfh
                  WHERE plnnr EQ t_plpo_aux-plnnr
                    AND plnkn EQ t_plpo_aux-plnkn
                    AND loekz NE 'X'.
                IF NOT t_plfh[] IS INITIAL.
                  SORT t_plfh[] ASCENDING BY objty objid.
                  DELETE ADJACENT DUPLICATES FROM t_plfh[] COMPARING objty objid.
                  IF NOT t_plfh[] IS INITIAL.
                    LOOP AT t_plfh.
                      SELECT SINGLE *
                        FROM crvm_b
                        WHERE objty EQ t_plfh-objty
                          AND objid EQ t_plfh-objid.
                      IF sy-subrc EQ 0.
                        MOVE crvm_b-matnr TO v_ferramenta.
                      ENDIF.
                    ENDLOOP.
                  ENDIF.
                ENDIF.
              ENDIF.
            ENDIF.
          ENDLOOP.
        ENDIF.
      ENDIF.
    ENDIF.
***Inserção - Hermiro Carvalho - INC 1732786 - 23/10/2014 - Início
  ELSE.
    CLEAR: p_beskz.
    SELECT SINGLE beskz
      FROM marc
      INTO p_beskz
      WHERE matnr EQ p_matnr
        AND werks EQ p_werks.
    IF p_beskz EQ 'F' OR p_beskz EQ 'X'.
      p_ok = 'X'.
    ENDIF.
***Inserção - Hermiro Carvalho - INC 1732786 - 23/10/2014 - Fim
  ENDIF.

ENDFORM.                    " check_MAT_CONFIGURADO

*&---------------------------------------------------------------------*
*&      Form  check_MAT_CONFIGURAVEL
*&---------------------------------------------------------------------*
FORM check_mat_configuravel  USING  p_klart TYPE ausp-klart
                                      p_kdauf TYPE vbap-vbeln
                                      p_kdpos TYPE vbap-posnr
                             CHANGING p_ok
                                      p_vl_matcli.

  CLEAR: t_carac.
  FREE:  t_carac.

  CALL FUNCTION 'ZRNSDF_SEARCH_COVD_MAE'
    EXPORTING
      ordem_venda              = p_kdauf
      item_ordem               = p_kdpos
      idioma                   = sy-langu
      classe                   = p_klart
    TABLES
      itab_ov                  = t_carac
    EXCEPTIONS
      not_found                = 1
      caracteristica_not_found = 2
      ordem_venda_is_missing   = 3
      idioma_is_missing        = 4
      OTHERS                   = 5.

  LOOP AT t_carac WHERE ( atnam = 'SD_ROTEIRO_PRODUCAO1' OR  atnam = 'SD_ROTEIRO_PRODUCAO2'  OR
                          atnam = 'SD_ROTEIRO_PRODUCAO3' OR  atnam = 'SD_ROTEIRO_PRODUCAO4'  OR
                          atnam = 'SD_ROTEIRO_PRODUCAO5' OR  atnam = 'SD_ROTEIRO_PRODUCAO6' ).
    p_ok = 'X'.       "Indica que possui centro de trabalho
    EXIT.
  ENDLOOP.

  READ TABLE t_carac WITH KEY atnam = 'SD_MATERIAL_CLIENTE'.
  IF sy-subrc EQ 0.
    p_vl_matcli = t_carac-resul.
    SHIFT p_vl_matcli LEFT DELETING LEADING space.
  ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Início
*  IF p_vl_matcli IS INITIAL.
*    READ TABLE t_carac WITH KEY atnam = 'SD_NUM_MAT1'.
*    IF sy-subrc EQ 0.
*      p_vl_matcli = t_carac-resul.
*      SHIFT p_vl_matcli LEFT DELETING LEADING space.
*    ENDIF.
*  ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Fim
ENDFORM.                    " CHECK_MAT_CONFIGURAVEL

*&---------------------------------------------------------------------*
*&      Form  F_SELECIONA_CRHD
*&---------------------------------------------------------------------*
FORM f_seleciona_crhd .

  CLEAR: t_crhd[].
  SELECT objid arbpl
     INTO TABLE t_crhd
     FROM crhd
* Projeto Unir2 - Fabiana - 28.08.2009 - Início
*     WHERE arbpl IN s_cent_t.
     WHERE arbpl IN s_cent_t
       AND verwe NE '0005'.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Início
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
*       AND werks IN p_centro.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim

* Projeto Unir2 - Fabiana - 28.08.2009 - Fim

  IF sy-subrc = 0.
    SORT t_crhd BY arbpl.
  ENDIF.

ENDFORM.                    " F_SELECIONA_CRHD

*&---------------------------------------------------------------------*
*&      Form  F_SALDO_MCHB
*&---------------------------------------------------------------------*
FORM f_saldo_mchb .

  DATA: vl_tabix LIKE sy-tabix.

  DATA: t_ov_separa  TYPE TABLE OF zrnppt_ov_separa,
        wa_ov_separa TYPE zrnppt_ov_separa.

  DATA: BEGIN OF t_zpmvm OCCURS 0,
          zmatnr    LIKE zrnppt_pmvm-zmatnr,
          zcharg    LIKE zrnppt_pmvm-zcharg,
          zqtpeca   LIKE zrnppt_pmvm-zqtpeca,
          zqtreal   LIKE zrnppt_pmvm-zqtreal,
          zqtreg    LIKE zrnppt_pmvm-zqtreg,
          zordvenda LIKE zrnppt_pmvm-zordvenda,
          zitem     LIKE zrnppt_pmvm-zitem,
          znumpp    LIKE zrnppt_pmvm-znumpp,
* Bruno Quadros - CSC - 29.07.2011 - Inicio
          zdatreg   LIKE zrnppt_pmvm-zdatreg,
* Bruno Quadros - CSC - 29.07.2011 - Fim
        END OF t_zpmvm.

  DATA  lv_qtd_pecas  TYPE p.

  DATA: t_lote   LIKE zrnmme_poa_caract_lo OCCURS 0 WITH HEADER LINE,
        lv_atinn TYPE cabn-atinn.

  DATA: t_melhoria_aux    TYPE TABLE OF ty_melhoria.

*RDM 257471
  DATA: BEGIN OF lit_expedi_cen OCCURS 0,
          werks TYPE mchb-werks,
        END OF lit_expedi_cen.
*RDM 257471

  "  IF t_melhoria[] IS NOT INITIAL.

**    SELECT werks lgort matnr charg clabs ersda cinsm
**      INTO TABLE t_mchb
**      FROM mchb
*** Bruno Quadros - CSC - 25.04.2011 - Inicio
*****Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Início
***    FOR ALL ENTRIES IN t_saida
***    WHERE matnr = t_saida-matnr
**      FOR ALL ENTRIES IN t_melhoria
**      WHERE matnr = t_melhoria-matnr
*****Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Fim
***     "AND werks = p_centro-low
*** Bruno Quadros - CSC - 25.04.2011 - Fim
**        AND ( clabs GT 0 OR cinsm GT 0 ).

  "  ENDIF.

  t_melhoria_aux[] = t_melhoria[].

  SORT t_melhoria_aux BY matnr.

  DELETE ADJACENT DUPLICATES FROM t_melhoria_aux COMPARING matnr.

*  LOOP AT t_melhoria INTO  wa_melhoria.
  LOOP AT t_melhoria_aux INTO  wa_melhoria.

    SELECT werks lgort matnr charg clabs ersda cinsm cspem
    APPENDING TABLE t_mchb
    FROM mchb
    WHERE matnr = wa_melhoria-matnr
* Início Sandra H. RDM 324995 - 01/25 - acrescentando lotes bloqueados
*    AND ( clabs GT 0 OR cinsm GT 0 ).
     AND ( clabs GT 0 OR cinsm GT 0 OR cspem GT 0 ).
* Fim Sandra H. RDM 324995 - 01/25.
  ENDLOOP.

* Início Sandra H. RDM 324995 - 01/25 - acrescentando lotes bloqueados
  LOOP AT t_mchb ASSIGNING FIELD-SYMBOL(<fs_mchb>).

    IF <fs_mchb>-cspem IS NOT INITIAL.
      <fs_mchb>-clabs    = <fs_mchb>-cspem.
    ENDIF.

  ENDLOOP.
* Fim Sandra H. RDM 324995 - 01/25

  "Início-Insert-UZ97768-INC000000118848 30 10 2024
** Sandra H - 03/01/25
*  DATA: r_werks TYPE RANGE OF mchb-werks WITH HEADER LINE,
*        r_matnr TYPE RANGE OF mchb-matnr WITH HEADER LINE.
*
*  DATA: r_mot_bloqueado TYPE RANGE OF mchb-kzics WITH HEADER LINE.
*
*  DATA: ls_mchb LIKE LINE OF t_mchb.
*
*  r_mot_bloqueado = 'IEQ'.
*  r_mot_bloqueado-low = '2'.
*  APPEND r_mot_bloqueado.
*
*  r_mot_bloqueado = 'IEQ'.
*  r_mot_bloqueado-low = '3'.
*  APPEND r_mot_bloqueado.
*
*  r_mot_bloqueado = 'IEQ'.
*  r_mot_bloqueado-low = '4'.
*  APPEND r_mot_bloqueado.
*
*  r_mot_bloqueado = 'IEQ'.
*  r_mot_bloqueado-low = '5'.
*  APPEND r_mot_bloqueado.
*
*  r_mot_bloqueado = 'IEQ'.
*  r_mot_bloqueado-low = '7'.
*  APPEND r_mot_bloqueado.
*
*  r_mot_bloqueado = 'IEQ'.
*  r_mot_bloqueado-low = '15'.
*  APPEND r_mot_bloqueado.
*
*  REFRESH t_mcha_mch1_mchb[].
*
*  REFRESH: r_werks,
*           r_matnr.
*
*  LOOP AT t_melhoria_aux INTO  wa_melhoria.
*
*    r_werks = 'IEQ'.
*    r_werks-low = wa_melhoria-werks.
*    APPEND r_werks.
*
*    r_matnr = 'IEQ'.
*    r_matnr-low = wa_melhoria-matnr.
*    APPEND r_matnr.
*
*  ENDLOOP.
*
*  SORT: r_werks[] BY low,
*        r_matnr[] BY low.
*
*  DELETE ADJACENT DUPLICATES FROM r_werks[] COMPARING low.
*  DELETE ADJACENT DUPLICATES FROM r_matnr[] COMPARING low.
*
*  cl_salv_bs_runtime_info=>set( display  = abap_false
*                                metadata = abap_false
*                                data     = abap_true ).
*
*  SUBMIT aqfkmm==========mcha_mch1_mchb  WITH pcentro  IN r_werks "• Centro
*                                         WITH sp$00001 IN r_matnr "• Material
*                                         AND RETURN.
*
*  TRY.
*      cl_salv_bs_runtime_info=>get_data_ref( IMPORTING r_data = DATA(lobj_data) ).
*      ASSIGN lobj_data->* TO FIELD-SYMBOL(<lft_data>).
*    CATCH cx_salv_bs_sc_runtime_info.
*  ENDTRY.
*
*  cl_salv_bs_runtime_info=>clear_all( ).
*
*  IF <lft_data> IS ASSIGNED.
*    t_mcha_mch1_mchb[] = CORRESPONDING #( <lft_data> ).
*  ENDIF.
*
*  LOOP AT t_mcha_mch1_mchb[] INTO DATA(ls_mcha_mch1_mchb).
*
*    READ TABLE t_mchb TRANSPORTING NO FIELDS WITH KEY charg = ls_mcha_mch1_mchb-mch1-charg
*                                                      lgort = ls_mcha_mch1_mchb-mchb-lgort
*                                                      werks = ls_mcha_mch1_mchb-mchb-werks
*                                                      matnr = ls_mcha_mch1_mchb-mch1-matnr.
*    IF sy-subrc NE 0.
*      ls_mchb-matnr    = ls_mcha_mch1_mchb-mch1-matnr.
*      ls_mchb-charg    = ls_mcha_mch1_mchb-mch1-charg.
*      ls_mchb-lgort    = ls_mcha_mch1_mchb-mchb-lgort.
*      ls_mchb-werks    = ls_mcha_mch1_mchb-mchb-werks.
*      IF ls_mcha_mch1_mchb-mchb-clabs IS NOT INITIAL.
*        ls_mchb-clabs    = ls_mcha_mch1_mchb-mchb-clabs.
*      ELSE.
*        ls_mchb-clabs    = ls_mcha_mch1_mchb-mchb-cspem.
*      ENDIF.
*      ls_mchb-ersda    = ls_mcha_mch1_mchb-mch1-ersda.
*      ls_mchb-cinsm    = ls_mcha_mch1_mchb-vs_peso_bruto.
*      APPEND ls_mchb TO t_mchb[].
*    ENDIF.
*
*  ENDLOOP.
  "Fim-Insert-UZ97768-INC000000118848 30 10 2024
** Sandra H. 03/01/2025

*  IF sy-subrc = 0.
  IF t_mchb[] IS NOT INITIAL.
***Inserção - Hermiro Carvalho - RDM 196192 - 21/06/2018 - Início
    SORT t_mchb BY werks lgort matnr charg clabs ersda cinsm cspem.
    DELETE ADJACENT DUPLICATES FROM t_mchb COMPARING werks lgort matnr charg clabs ersda cinsm cspem.
***Inserção - Hermiro Carvalho - RDM 196192 - 21/06/2018 - Fim
* Selecionar na tabela PMVM o peso real
    CLEAR: t_zpmvm[].
    SELECT zmatnr zcharg zqtpeca zqtreal zqtreg zordvenda zitem znumpp
* Bruno Quadros - CSC - 29.07.2011 - Inicio
           zdatreg
* Bruno Quadros - CSC - 29.07.2011 - Fim
      INTO TABLE t_zpmvm
      FROM zrnppt_pmvm
      FOR ALL ENTRIES IN t_mchb
      WHERE zmatnr = t_mchb-matnr
        AND zcharg = t_mchb-charg
        AND zcodmov = '101'
        AND zflag <> 'x'.
    SORT t_zpmvm BY zmatnr zcharg.

***Inserção - Hermiro Carvalho - PRJ01313 - 26/11/2015 - Início
    CLEAR: t_ov_separa[].
    SELECT werks lgort matnr charg kalab edatu zqtpeca vbeln posnr
              FROM zrnppt_ov_separa
              INTO CORRESPONDING FIELDS OF TABLE t_ov_separa
              FOR ALL ENTRIES IN t_mchb
             WHERE matnr  = t_mchb-matnr
               AND werks  = t_mchb-werks
               AND lgort  = t_mchb-lgort
               AND charg  = t_mchb-charg
               AND vbeln  NE space   " Sandra H. RDM 324995 - 01/25
               AND ( status <> 'X' AND status <> 'C' ).  "Status <> Fornecida ou Status <> Cancelada
    LOOP AT t_ov_separa INTO wa_ov_separa.
      t_zpmvm-zmatnr    = wa_ov_separa-matnr.
      t_zpmvm-zcharg    = wa_ov_separa-charg.
***Inserção - Hermiro Carvalho - RDM 160097 - 16/05/2016 - Início
      CASE wa_ov_separa-zqtpeca.
        WHEN 0.
          t_zpmvm-zqtpeca = 0.
        WHEN OTHERS.
          t_zpmvm-zqtpeca   = wa_ov_separa-zqtpeca / 1000.
      ENDCASE.
***Inserção - Hermiro Carvalho - RDM 160097 - 16/05/2016 - Fim
      t_zpmvm-zqtreal   = wa_ov_separa-kalab.
      t_zpmvm-zqtreg    = wa_ov_separa-kalab.
      t_zpmvm-zordvenda = wa_ov_separa-vbeln.
      t_zpmvm-zitem     = wa_ov_separa-posnr.
      t_zpmvm-znumpp    =  ''.
      APPEND t_zpmvm.
      CLEAR t_zpmvm.
    ENDLOOP.

    SORT t_zpmvm BY zmatnr zcharg.
* ----------------------------------------------------------------------
* INICIO: 09mar2017 RGC8096 RDM 176853
* ----------------------------------------------------------------------
***Inserção - SAndra H. 28/03/2017 - RDM 178034 - Início
    IF t_mchb[] IS NOT INITIAL.
      SELECT matnr werks strgr minbe bstma shzet lgpro sobsl
            APPENDING TABLE t_marc
            FROM marc
            FOR ALL ENTRIES IN t_mchb
            WHERE matnr = t_mchb-matnr
              AND werks = t_mchb-werks.
    ENDIF.

    SORT t_marc BY matnr werks.
    DELETE ADJACENT DUPLICATES FROM t_marc COMPARING ALL FIELDS.
***Inserção - SAndra H. 28/03/2017 - RDM 178034 - Fim

    LOOP AT t_mchb.
      READ TABLE   t_zpmvm WITH KEY zmatnr = t_mchb-matnr
                                    zcharg = t_mchb-charg.
*     IF sy-subrc NE 0.
      IF sy-subrc NE 0 AND t_marc-strgr NE '70'. "SAndra H. 28/03/2017 - RDM 178034 - Início

        t_zpmvm-zmatnr    = t_mchb-matnr.
        t_zpmvm-zcharg    = t_mchb-charg.
        t_zpmvm-zordvenda = space.
        t_zpmvm-zitem     = space.
        t_zpmvm-znumpp    = space.

        READ TABLE t_mara WITH KEY matnr = t_mchb-matnr BINARY SEARCH.

        IF t_mara-meins = 'PC' OR t_mara-meins = 'ST'.

          PERFORM trata_peca USING t_mchb-matnr t_saida-pn_interno.

          IF t_mchb-clabs IS NOT INITIAL.

            t_zpmvm-zqtpeca   = t_mchb-clabs.
            t_zpmvm-zqtreal   = v_qtde * t_mchb-peca. "peso convertido.

          ELSEIF t_mchb-cinsm IS NOT INITIAL.

            t_zpmvm-zqtpeca   = t_mchb-cinsm.
            t_zpmvm-zqtreal   =  v_qtde * t_mchb-peca. "peso convertido

          ENDIF.

        ELSE .

* 10mar2017 RGC8096
          IF t_mara-meins = 'TO' OR t_mara-meins = 'TN'.

            CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
              EXPORTING
                input  = 'SD_QUANTIDADE_PECAS'
              IMPORTING
                output = lv_atinn.

            REFRESH t_lote.

            CALL FUNCTION 'ZRNMMF_SEARCH_CLOT'
              EXPORTING
                matnr                     = t_mchb-matnr
                klart                     = '023'
                charg                     = t_mchb-charg
                idioma                    = sy-langu
                caracteristica            = lv_atinn
              TABLES
                itab_lot                  = t_lote
              EXCEPTIONS
                matnr_not_found           = 1
                matnr_or_mara_not_found   = 2
                caracteristicas_not_found = 3
                classe_not_found          = 4
                OTHERS                    = 5.

            IF sy-subrc EQ 0.

              READ TABLE t_lote INDEX 1.

              lv_qtd_pecas = t_lote-resul.

              t_zpmvm-zqtpeca = ( lv_qtd_pecas / 1000 ).

            ENDIF.

            IF t_mchb-clabs IS NOT INITIAL.
              t_zpmvm-zqtreg   = t_mchb-clabs.
            ELSEIF t_mchb-cinsm IS NOT INITIAL.
              t_zpmvm-zqtreg    = t_mchb-cinsm.
            ENDIF.

            APPEND t_zpmvm.
            CLEAR t_zpmvm.

          ENDIF.

        ENDIF.
      ENDIF.
    ENDLOOP.

    SORT t_zpmvm BY zmatnr zcharg.

* ----------------------------------------------------------------------
* FIM: 09mar2017 RGC8096 RDM 176853
* ----------------------------------------------------------------------

*    IF sy-subrc = 0.
*    IF t_zpmvm[] IS NOT INITIAL.
***Inserção - Hermiro Carvalho - PRJ01313 - 26/11/2015 - Fim
    LOOP AT t_mchb.

      v_tabix = sy-tabix.

      READ TABLE t_mara WITH KEY matnr = t_mchb-matnr
                                 BINARY SEARCH.
      IF sy-subrc = 0.

        READ TABLE t_zrnppt_prev WITH KEY matnr = t_mchb-matnr
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
                                          werks = t_mchb-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
        IF sy-subrc = 0.
          t_mchb-bismt = t_zrnppt_prev-bismt.
        ENDIF.

        READ TABLE t_zpmvm WITH KEY zmatnr = t_mchb-matnr
                                    zcharg = t_mchb-charg
                                    BINARY SEARCH.
        IF sy-subrc IS INITIAL.

          t_mchb-ov     = t_zpmvm-zordvenda.
          t_mchb-it     = t_zpmvm-zitem.
          t_mchb-znumpp = t_zpmvm-znumpp.
          t_mchb-zdatreg = t_zpmvm-zdatreg.

          IF t_mara-meins = 'PC' OR t_mara-meins = 'ST'.
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Início
*            IF t_mchb-clabs IS NOT INITIAL.
            IF t_mchb-clabs IS NOT INITIAL AND t_mchb-lgort <> '0096'.
              t_mchb-clabs = t_zpmvm-zqtreal.
*            ELSEIF t_mchb-cinsm IS NOT INITIAL.
            ELSEIF t_mchb-cinsm IS NOT INITIAL AND t_mchb-lgort = '0096'.
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Fim
              t_mchb-cinsm = t_zpmvm-zqtreal.
            ENDIF.
          ELSEIF t_mara-meins = 'TO' OR t_mara-meins = 'TN'.
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Início
*            IF t_mchb-clabs IS NOT INITIAL.
            IF t_mchb-clabs IS NOT INITIAL AND t_mchb-lgort <> '0096'.
              t_mchb-clabs = t_zpmvm-zqtreg.
*            ELSEIF t_mchb-cinsm IS NOT INITIAL.
            ELSEIF t_mchb-cinsm IS NOT INITIAL AND t_mchb-lgort = '0096'.
              t_mchb-cinsm = t_zpmvm-zqtreg.
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Fim
            ENDIF.
          ENDIF.

          t_mchb-peca  = t_zpmvm-zqtpeca * 1000.
          MODIFY t_mchb INDEX v_tabix.

        ELSE. "Produção Externa sem o ZPETIQ

          IF t_mara-meins = 'PC' OR t_mara-meins = 'ST'.

            PERFORM trata_peca USING t_mchb-matnr
                                     ''.
            IF t_mchb-clabs IS NOT INITIAL.
              t_mchb-peca = t_mchb-clabs.
              t_mchb-clabs  =  v_qtde * t_mchb-peca. "peso convertido
            ELSEIF t_mchb-cinsm IS NOT INITIAL.
              t_mchb-peca = t_mchb-cinsm.
              t_mchb-cinsm  =  v_qtde * t_mchb-peca. "peso convertido
            ENDIF.

          ELSEIF t_mara-meins = 'TO' OR t_mara-meins = 'TN'.

            CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
              EXPORTING
                input  = 'SD_QUANTIDADE_PECAS'
              IMPORTING
                output = lv_atinn.

            REFRESH t_lote.
            CALL FUNCTION 'ZRNMMF_SEARCH_CLOT'
              EXPORTING
                matnr                     = t_mchb-matnr
                klart                     = '023'
                charg                     = t_mchb-charg
                idioma                    = sy-langu
                caracteristica            = lv_atinn
              TABLES
                itab_lot                  = t_lote
              EXCEPTIONS
                matnr_not_found           = 1
                matnr_or_mara_not_found   = 2
                caracteristicas_not_found = 3
                classe_not_found          = 4
                OTHERS                    = 5.
            IF sy-subrc EQ 0.
              READ TABLE t_lote INDEX 1.
              lv_qtd_pecas = t_lote-resul.
              t_mchb-peca = ( lv_qtd_pecas / 1000 ).
            ENDIF.

          ENDIF.

          MODIFY t_mchb INDEX v_tabix.

        ENDIF.

      ENDIF.

    ENDLOOP.

  ENDIF.

  IF t_mchb[] IS NOT INITIAL.
*
*    SELECT matnr werks strgr minbe
*      APPENDING TABLE t_marc
*      FROM marc
*      FOR ALL ENTRIES IN t_mchb
*      WHERE matnr = t_mchb-matnr
** Bruno Quadros - CSC - 25.04.2011 - Inicio
*        AND werks = t_mchb-werks.
*
*    SORT t_marc BY matnr werks.
*    DELETE ADJACENT DUPLICATES FROM t_marc COMPARING ALL FIELDS.
** Bruno Quadros - CSC - 25.04.2011 - Fim

* Atribui expedição a tabela de saida
    SORT t_mchb BY matnr matcli ferramenta.
  ENDIF.

  LOOP AT t_saida.
    v_tabix = sy-tabix.

    CLEAR: v_expedi, v_expedi_aux, v_mat_cq, v_exp_inter, t_marc,
           v_peca_expedic, v_peca_mat_cq, v_peca_exp_int, v_peca_expedic_aux.

*RDM 257471
    CLEAR: lit_expedi_cen[], lit_expedi_cen.
*RDM 257471

    READ TABLE t_marc WITH KEY matnr = t_saida-matnr
*** Início Alteração Sandra H - maio/2017 -  RDI 2351247
*                              werks = t_saida-wrk02
                               werks = t_saida-werks
                               BINARY SEARCH.
*** Fim Alteração Sandra H - maio/2017 -  RDI 2351247
    IF sy-subrc = 0.

      IF t_marc-strgr NE '20' AND
         t_marc-strgr NE '25'.

        LOOP AT t_mchb WHERE matnr = t_saida-matnr.

          READ TABLE t_zpmvm WITH KEY zcharg = t_mchb-charg.
          IF sy-subrc IS INITIAL.
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Início
*            IF t_mchb-clabs IS NOT INITIAL.
            IF t_mchb-clabs IS NOT INITIAL AND t_mchb-lgort <> '0096'.
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Fim
              IF t_saida-matnr GE '50000000' AND t_saida-matnr LE '54999999'.
                ADD t_mchb-clabs TO v_exp_inter.
                ADD t_mchb-peca  TO v_peca_exp_int.
              ELSE.
                IF t_mchb-werks = t_saida-werks.
                  ADD t_mchb-clabs TO v_expedi.
                  ADD t_mchb-peca TO v_peca_expedic.
                ELSE.
                  ADD t_mchb-clabs TO v_expedi_aux.
                  ADD t_mchb-peca TO v_peca_expedic_aux.
*RDM 257471
                  lit_expedi_cen-werks = t_mchb-werks.
                  APPEND lit_expedi_cen.
*RDM 257471
                ENDIF.
              ENDIF.

            ELSEIF t_mchb-cinsm IS NOT INITIAL OR t_mchb-lgort EQ '0096'. "Sandra Hernandez - RDI 2305759 - 15/03/2017
***Inserção - Sandra H. - RDI 2305759 - 15/03/2017 - Início
              IF t_mchb-cinsm IS NOT INITIAL.
                ADD t_mchb-cinsm TO v_mat_cq.
                ADD t_mchb-peca  TO v_peca_mat_cq.
              ENDIF.
***Fim - Sandra H. - RDI 2305759 - 15/03/2017 - Fim
              IF t_mchb-lgort EQ '0096'."*****Hermiro 09/02/2017
                ADD t_mchb-clabs TO v_mat_cq.
                ADD t_mchb-peca  TO v_peca_mat_cq.
              ENDIF."*****Hermiro 09/02/2017
***Inserção - Hermiro Carvalho - RDM 175668 - 08/02/2017 - Fim
            ENDIF.

          ELSE. "Produção Externa sem o ZPETIQ
****Inserção - Sandra Hernandez- RDM 178034 - Inicio
*            IF t_marc-strgr NE '70'.  " quando venda de MP in-natura estoque em expedição apenas o que está no OV_SEPARA
****FIM - Sandra Hernandez- RDM 178034 - Inicio
            READ TABLE t_mara WITH KEY matnr = t_saida-matnr
                                       BINARY SEARCH.
            IF sy-subrc IS NOT INITIAL.
              SELECT SINGLE meins INTO t_mara-meins
                FROM mara
                WHERE matnr = t_saida-matnr.
            ENDIF.
            IF sy-subrc IS INITIAL.

              IF t_mara-meins = 'PC' OR t_mara-meins = 'ST'.
                IF t_mchb-clabs IS NOT INITIAL AND t_mchb-lgort <> '0096'."*****Hermiro 09/02/2017
***Inserção - Sandra Hernandez- RDM 178034 - Inicio
                  IF t_marc-strgr NE '70'.  " quando venda de MP in-natura estoque em expedição apenas o que está no OV_SEPARA
***FIM - Sandra Hernandez- RDM 178034 - Inicio

                    IF t_saida-matnr GE '50000000' AND t_saida-matnr LE '54999999'.
                      ADD t_mchb-clabs TO v_exp_inter.
                      ADD t_mchb-peca TO v_peca_exp_int.
                    ELSE.

                      IF t_mchb-werks = t_saida-werks.
                        ADD t_mchb-clabs TO v_expedi.
                        ADD t_mchb-peca TO v_peca_expedic.
                      ELSE.
                        ADD t_mchb-clabs TO v_expedi_aux.
                        ADD t_mchb-peca TO v_peca_expedic_aux.
*RDM 257471
                        lit_expedi_cen-werks = t_mchb-werks.
                        APPEND lit_expedi_cen.
*RDM 257471
                      ENDIF.

                    ENDIF.
                  ENDIF. " Sandra Hernandez- RDM 178034

                ELSEIF t_mchb-cinsm IS NOT INITIAL.
                  ADD t_mchb-peca TO v_peca_mat_cq.
                  ADD t_mchb-cinsm TO v_peca_mat_cq.
                ENDIF.

              ELSEIF t_mara-meins = 'TO' OR t_mara-meins = 'TN'.

                IF t_mchb-clabs IS NOT INITIAL AND t_mchb-lgort <> '0096'."*****Hermiro 09/02/2017

                  IF t_marc-strgr NE '70'. " Venda in-natura - Sandra Hernandez- RDM 178034
                    IF t_saida-matnr GE '50000000' AND t_saida-matnr LE '54999999'.
                      ADD t_mchb-clabs TO v_exp_inter.
                      ADD t_mchb-peca TO v_peca_exp_int.
                    ELSE.

                      IF t_mchb-werks = t_saida-werks.
                        ADD t_mchb-clabs TO v_expedi.
                        ADD t_mchb-peca TO v_peca_expedic.
                      ELSE.
                        ADD t_mchb-clabs TO v_expedi_aux.
                        ADD t_mchb-peca TO v_peca_expedic_aux.
*RDM 257471
                        lit_expedi_cen-werks = t_mchb-werks.
                        APPEND lit_expedi_cen.
*RDM 257471
                      ENDIF.

                    ENDIF.
                  ENDIF. " Venda in-natura - Sandra Hernandez- RDM 178034
                ELSEIF t_mchb-cinsm IS NOT INITIAL.

                  ADD t_mchb-cinsm TO v_mat_cq.
                  ADD t_mchb-peca  TO v_peca_mat_cq.
***Inserção - Hermiro Carvalho - Hermiro Carvalho - RDM 175668 - 09/02/2017 - Início
                ELSEIF t_mchb-clabs IS NOT INITIAL AND t_mchb-lgort = '0096'.
                  ADD t_mchb-clabs TO v_mat_cq.
                  ADD t_mchb-peca  TO v_peca_mat_cq.
***Inserção - Hermiro Carvalho - Hermiro Carvalho - RDM 175668 - 09/02/2017 - Fim
                ENDIF.
              ENDIF.
            ENDIF.

          ENDIF.
***Inserção - Sandra Hernandez- RDM 178034 - Inicio
*          ENDIF.
***FIM - Sandra Hernandez- RDM 178034 - Inicio
        ENDLOOP.

      ELSE.   "Estrategia 20 ou 25

        LOOP AT t_mchb WHERE matnr      = t_saida-matnr
                         AND matcli     = t_saida-codigo
                         AND ferramenta = t_saida-ferramenta.

          READ TABLE t_zpmvm WITH KEY zcharg = t_mchb-charg.
          IF sy-subrc IS INITIAL.

            IF t_mchb-clabs IS NOT INITIAL.
              IF t_saida-matnr GE '50000000' AND t_saida-matnr LE '54999999'.
                ADD t_mchb-clabs TO v_exp_inter.
                ADD t_mchb-peca  TO v_peca_exp_int.
              ELSE.
                IF t_mchb-werks = t_saida-werks.
                  ADD t_mchb-clabs TO v_expedi.
                  ADD t_mchb-peca TO v_peca_expedic.
                ELSE.
                  ADD t_mchb-clabs TO v_expedi_aux.
                  ADD t_mchb-peca TO v_peca_expedic_aux.
*RDM 257471
                  lit_expedi_cen-werks = t_mchb-werks.
                  APPEND lit_expedi_cen.
*RDM 257471
                ENDIF.
              ENDIF.
            ELSEIF t_mchb-cinsm IS NOT INITIAL.
              IF t_mchb-lgort EQ '0096'."*****Hermiro 09/02/2017
                ADD t_mchb-cinsm TO v_mat_cq.
                ADD t_mchb-peca  TO v_peca_mat_cq.
              ENDIF."*****Hermiro 09/02/2017
            ENDIF.

          ELSE. "Produção Externa sem o ZPETIQ

            READ TABLE t_mara WITH KEY matnr = t_saida-matnr
                                       BINARY SEARCH.
            IF sy-subrc IS NOT INITIAL.
              SELECT SINGLE meins INTO t_mara-meins
                FROM mara
                WHERE matnr = t_saida-matnr.
            ENDIF.

            IF sy-subrc IS INITIAL.
              IF t_mara-meins = 'PC' OR t_mara-meins = 'ST'.

                IF t_mchb-clabs IS NOT INITIAL.
                  IF t_saida-matnr GE '50000000' AND t_saida-matnr LE '54999999'.
                    ADD t_mchb-clabs TO v_exp_inter.
                    ADD t_mchb-peca TO v_peca_exp_int.
                  ELSE.

                    IF t_mchb-werks = t_saida-werks.
                      ADD t_mchb-clabs TO v_expedi.
                      ADD t_mchb-peca TO v_peca_expedic.
                    ELSE.
                      ADD t_mchb-clabs TO v_expedi_aux.
                      ADD t_mchb-peca TO v_peca_expedic_aux.
*RDM 257471
                      lit_expedi_cen-werks = t_mchb-werks.
                      APPEND lit_expedi_cen.
*RDM 257471
                    ENDIF.
                  ENDIF.

                ELSEIF t_mchb-cinsm IS NOT INITIAL.

                  IF t_mchb-lgort EQ '0096'."*****Hermiro 09/02/2017
                    ADD t_mchb-peca TO v_peca_mat_cq.
                    ADD t_mchb-cinsm TO v_peca_mat_cq.
                  ENDIF."*****Hermiro 09/02/2017
                ENDIF.

              ELSEIF t_mara-meins = 'TO' OR t_mara-meins = 'TN'.

                IF t_mchb-clabs IS NOT INITIAL AND t_mchb-lgort <> '0096'."*****Hermiro 09/02/2017

                  IF t_saida-matnr GE '50000000' AND t_saida-matnr LE '54999999'.
                    ADD t_mchb-clabs TO v_exp_inter.
                    ADD t_mchb-peca TO v_peca_exp_int.
                  ELSE.

                    IF t_mchb-werks = t_saida-werks.
                      ADD t_mchb-clabs TO v_expedi.
                      ADD t_mchb-peca TO v_peca_expedic.
                    ELSE.
                      ADD t_mchb-clabs TO v_expedi_aux.
                      ADD t_mchb-peca TO v_peca_expedic_aux.
*RDM 257471
                      lit_expedi_cen-werks = t_mchb-werks.
                      APPEND lit_expedi_cen.
*RDM 257471
                    ENDIF.

                  ENDIF.
                ELSEIF t_mchb-cinsm IS NOT INITIAL.

                  IF t_mchb-lgort EQ '0096'."*****Hermiro 09/02/2017
                    ADD t_mchb-cinsm TO v_mat_cq.
                    ADD t_mchb-peca  TO v_peca_mat_cq.
                  ENDIF."*****Hermiro 09/02/2017
                ENDIF.

              ENDIF.

            ENDIF.

          ENDIF.

        ENDLOOP.

      ENDIF.

      t_saida-exp_inter  = v_exp_inter.
      t_saida-mat_cq     = v_mat_cq.
      t_saida-expedicao  = v_expedi.
      t_saida-expedi_cen = v_expedi_aux.
      t_saida-analise_prev  = v_expedi + v_expedi_aux - t_saida-estoque_minimo.
      t_saida-expedicao_pec = v_peca_expedic.
      t_saida-exp_inter_pec = v_peca_exp_int.
      t_saida-mat_cq_pec    = v_peca_mat_cq.
      t_saida-analise_prev_pec = t_saida-expedicao_pec - t_saida-est_min_pec.
      t_saida-expedi_cen_pc = v_peca_expedic_aux.

*RDM 257471
      CLEAR t_saida-expedi_cen_det.

      SORT lit_expedi_cen BY werks ASCENDING.
      DELETE ADJACENT DUPLICATES FROM lit_expedi_cen COMPARING ALL FIELDS.
      LOOP AT lit_expedi_cen.
        IF t_saida-expedi_cen_det IS INITIAL.
          t_saida-expedi_cen_det = lit_expedi_cen.
        ELSE.
          CONCATENATE t_saida-expedi_cen_det lit_expedi_cen INTO t_saida-expedi_cen_det SEPARATED BY '-'.
        ENDIF.
      ENDLOOP.

      FREE lit_expedi_cen.
*RDM 257471

    ENDIF.

    IF NOT t_saida-estoque_minimo IS INITIAL.
      t_saida-analise = ( t_saida-analise_prev / t_saida-estoque_minimo ).
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
*    IF t_saida-peso_peca IS NOT INITIAL AND t_marc-minbe IS NOT INITIAL.
*
*      READ TABLE t_mara WITH KEY matnr = t_saida-matnr
*                                 BINARY SEARCH.
*      IF sy-subrc IS INITIAL.
*        IF t_marc IS NOT INITIAL.
*          IF t_mara-meins = 'PC'.
*            t_saida-abast_pec = t_marc-minbe.
*            t_saida-abast     = t_marc-minbe / t_saida-peso_peca.              " se esta em kg entao divide pelo peso da pc pra achar o PESO
*          ELSEIF t_mara-meins = 'TO' OR t_mara-meins = 'TN'.
*            t_saida-abast     = t_marc-minbe.
*            t_saida-abast_pec = ( t_marc-minbe * 1000 ) / t_saida-peso_peca.   "Se esta em to entao * por 1000 e / pelo peso da pc para a qtd peca
*          ENDIF.
*        ENDIF.
*      ENDIF.
*    ENDIF.
*    READ TABLE
    t_saida-abast     = t_saida-expedicao / 30  * ( t_saida-est_min_dias  + 5 ).
    t_saida-abast_pec = t_saida-expedicao_pec / 30  * ( t_saida-est_min_dias + 5 ).
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
    MODIFY t_saida INDEX v_tabix.

  ENDLOOP.

ENDFORM.                    " F_SALDO_MCHB

*&---------------------------------------------------------------------*
*&      Form  F_FATURADO
*&---------------------------------------------------------------------*
FORM f_faturado .

  DATA: t_vbap TYPE TABLE OF vbap WITH HEADER LINE.

  IF NOT t_saida[] IS INITIAL AND
     NOT t_melhoria[] IS INITIAL.

    SELECT vbeln erdat kunag fksto
      FROM vbrk
      INTO TABLE t_vbrk
      FOR ALL ENTRIES IN t_saida
      WHERE vbtyp = 'M'
        AND erdat IN p_period
        AND fktyp = 'L'
        AND bukrs = 'RNA1'
        AND kunag = t_saida-cod_cli
        AND fksto = ' '.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*        AND bupla = p_centro-low.
*        AND bupla = t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - I


    IF sy-subrc = 0.
      SORT t_vbrk BY vbeln.
    ENDIF.

    SELECT p~vbeln p~posnr p~aubel p~aupos p~matnr p~ntgew p~fkimg p~werks
           p~erdat k~xblnr
      INTO TABLE t_vbrp
      FROM vbrp AS p INNER JOIN vbrk AS k
        ON p~vbeln = k~vbeln
***Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Início
*      FOR ALL ENTRIES IN t_saida
*       WHERE p~matnr = t_saida-matnr
      FOR ALL ENTRIES IN t_melhoria
       WHERE p~matnr = t_melhoria-matnr
***Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Fim
*   Bruno Quadros - CSC - 25.04.2011 - Inicio
*         AND p~werks = p_centro-low
*         AND p~werks = t_saida-wrk02
*   Bruno Quadros - CSC - 25.04.2011 - Fim
         AND p~pstyv <> 'ZCFO'
         AND p~erdat IN p_period.

    SORT t_vbrp BY vbeln.

  ENDIF.

  IF sy-subrc = 0.

    LOOP AT t_vbrp.
      v_tabix = sy-tabix.
      READ TABLE t_vbrk WITH KEY vbeln = t_vbrp-vbeln
                                 BINARY SEARCH.
      IF sy-subrc = 0.
        t_vbrp-kunag  = t_vbrk-kunag.
        MODIFY t_vbrp INDEX v_tabix.
      ELSE.
        DELETE t_vbrp INDEX v_tabix.
      ENDIF.
    ENDLOOP.

    SELECT matnr werks strgr minbe bstma shzet lgpro sobsl
      APPENDING TABLE t_marc
      FROM marc
      FOR ALL ENTRIES IN t_vbrp
      WHERE matnr = t_vbrp-matnr
* Bruno Quadros - CSC - 25.04.2011 - Inicio
        AND werks = t_vbrp-werks.

    SORT t_marc BY matnr werks.
    DELETE ADJACENT DUPLICATES FROM t_marc COMPARING ALL FIELDS.
* Bruno Quadros - CSC - 25.04.2011 - Fim

    SORT t_vbrp BY matnr.

  ENDIF.

  IF t_vbrp[] IS NOT INITIAL.
    SELECT *
      FROM vbap
      INTO TABLE t_vbap
      FOR ALL ENTRIES IN t_vbrp[]
      WHERE vbeln = t_vbrp-aubel AND
            posnr = t_vbrp-aupos.
  ENDIF.

  LOOP AT t_vbrp.

    v_tabix = sy-tabix.

    CLEAR t_marc.
    READ TABLE t_marc WITH KEY matnr = t_vbrp-matnr
* Bruno Quadros - CSC - 25.04.2011 - Inicio
                               werks = t_vbrp-werks
                               BINARY SEARCH.
* Bruno Quadros - CSC - 25.04.2011 - Fim
    IF sy-subrc = 0.
      CASE t_marc-strgr.

        WHEN '40'
          OR '20'
          OR '70'.  " Inclusão Sandra H. 27/04/2017 - RDM 178034

          READ TABLE t_vbap WITH KEY vbeln = t_vbrp-aubel
                                     posnr = t_vbrp-aupos.
          IF sy-subrc = 0.
            t_vbrp-kdmat = t_vbap-kdmat.
            READ TABLE t_zrnppt_prev WITH  KEY matnr  = t_vbrp-matnr
                                               codigo = t_vbrp-kdmat.
          ELSE.
            READ TABLE t_zrnppt_prev WITH  KEY matnr = t_vbrp-matnr.
          ENDIF.

          IF sy-subrc EQ 0.
            t_vbrp-matcli = t_zrnppt_prev-codigo .
          ENDIF.

          MODIFY t_vbrp INDEX v_tabix.

        WHEN '25'.

          READ TABLE t_vbap WITH KEY vbeln = t_vbrp-aubel
                                     posnr = t_vbrp-aupos.

          IF sy-subrc = 0.
            t_vbrp-kdmat = t_vbap-kdmat.
*
            READ TABLE t_zrnppt_prev WITH  KEY  matnr  = t_vbrp-matnr
                                                kunnr  = t_vbrp-kunag
                                                codigo = t_vbrp-kdmat.
*
          ENDIF.

          IF sy-subrc EQ 0.
*            CONDENSE t_zrnppt_prev-codigo NO-GAPS.
            t_vbrp-matcli = t_zrnppt_prev-codigo.
          ENDIF.
          MODIFY t_vbrp INDEX v_tabix.

      ENDCASE.
    ENDIF.

  ENDLOOP.

* Insere Faturado na tabela de saida
  SORT t_vbrp BY matnr matcli kunag.

  LOOP AT t_saida.

    v_tabix = sy-tabix.

    READ TABLE t_marc WITH KEY matnr = t_saida-matnr
                               werks = t_saida-werks.
    IF t_marc-strgr = '25'.
      LOOP AT t_vbrp WHERE matnr  = t_saida-matnr
                       AND kdmat  = t_saida-codigo
                       AND kunag  = t_saida-cod_cli.

        IF t_vbrp-werks = t_saida-werks.
          t_saida-faturado1 = t_saida-faturado1 + t_vbrp-ntgew.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
****Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Início
*            t_saida-faturado1_pc = t_saida-faturado1_pc + t_vbrp-fkimg.
****Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Fim
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida-faturado1_pc = t_saida-faturado1_pc + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

        ELSE.
          t_saida-faturado1_cen = t_saida-faturado1_cen + t_vbrp-ntgew.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
****Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Início
*            t_saida-faturado1_cen_pc = t_saida-faturado1_cen_pc + t_vbrp-fkimg.
****Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Fim
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida-faturado1_cen_pc = t_saida-faturado1_cen_pc + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

        ENDIF.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Inicio
*        t_saida-ferramenta = t_vbrp-ferramenta.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Início
**        t_saida-faturado_pc = t_saida-faturado_pc + t_vbrp-fkimg.
***Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Fim
      ENDLOOP.

    ENDIF.

*    ENDIF.
* Inclusão - Rodrigo Soeiro - 12/2011 - Inicio
    IF t_marc-strgr EQ '20' .
* Fim da Inclusão

      LOOP AT t_vbrp WHERE matnr  = t_saida-matnr
                       AND matcli = t_saida-codigo
                       AND kunag  = t_saida-cod_cli.

        IF t_vbrp-werks = t_saida-werks.
          t_saida-faturado1 = t_saida-faturado1 + t_vbrp-ntgew.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
****Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Início
*            t_saida-faturado1_pc = t_saida-faturado1_pc + t_vbrp-fkimg.
****Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Fim
          v_pecas = t_vbrp-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida-faturado1_pc = t_saida-faturado1_pc + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim
        ELSE.
          t_saida-faturado1_cen = t_saida-faturado1_cen + t_vbrp-ntgew.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
****Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Início
*            t_saida-faturado1_cen_pc = t_saida-faturado1_cen_pc + t_vbrp-fkimg.
****Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Fim
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida-faturado1_cen_pc = t_saida-faturado1_cen_pc + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim
        ENDIF.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Início
*        t_saida-ferramenta = t_vbrp-ferramenta.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim
* Bruno Quadros - CSC - 12.04.2011 - Inicio
***Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Início
*        t_saida-faturado_pc = t_saida-faturado_pc + t_vbrp-fkimg.
***Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Fim
* Bruno Quadros - CSC - 12.04.2011 - Fim
      ENDLOOP.

    ELSE.
*******************preenche o range com o código do cliente caso a seja matnr estrategia 25
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Inicio
*      CLEAR: r_cod_cli, r_cod_cli[].
*
*      LOOP AT t_cod_cli WHERE sortl = t_saida-cliente.
*        r_cod_cli-sign = 'I'.
*        r_cod_cli-option = 'EQ'.
*        r_cod_cli-low = t_cod_cli-kunnr.
*        APPEND r_cod_cli.
*        CLEAR r_cod_cli.
*      ENDLOOP.
*******************
*
*      LOOP AT t_vbrp WHERE matnr = t_saida-matnr
*                       AND kunag IN r_cod_cli
*                       AND matcli = t_saida-codigo.

      LOOP AT t_vbrp WHERE matnr  = t_saida-matnr
                       AND matcli = t_saida-codigo
                       AND kunag  = t_saida-cod_cli.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim

        IF t_vbrp-werks = t_saida-werks.
          t_saida-faturado1 = t_saida-faturado1 + t_vbrp-ntgew.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
****Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Início
*            t_saida-faturado1_pc = t_saida-faturado1_pc + t_vbrp-fkimg.
****Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Fim
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida-faturado1_pc = t_saida-faturado1_pc + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim
        ELSE.
          t_saida-faturado1_cen = t_saida-faturado1_cen + t_vbrp-ntgew.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
****Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Início
*            t_saida-faturado1_cen_pc = t_saida-faturado1_cen_pc + t_vbrp-fkimg.
****Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Fim
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida-faturado1_cen_pc = t_saida-faturado1_cen_pc + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim
        ENDIF.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Início
****Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Início
*        t_saida-ferramenta = t_vbrp-ferramenta.
****Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Fim
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim
* Bruno Quadros - CSC - 12.04.2011 - Inicio
*        t_saida-faturado_pc = t_saida-faturado_pc + t_vbrp-fkimg.
* Bruno Quadros - CSC - 12.04.2011 - Fim
      ENDLOOP.

    ENDIF.

** Início da Alteração - Sandra H - 28.10.2025 - projeto
    PERFORM: zf_qualifica_demanda.
** Fim da Alteração - Sandra H - 28.10.2025 - projeto

    MODIFY t_saida INDEX v_tabix.

*    CLEAR t_saida.
* Inicio RDM 324238 - Sandra H.
    CLEAR: t_saida,  v_pecas.
* Fim RDM 324238 - Sandra H.

  ENDLOOP.

ENDFORM.                    " F_FATURADO

*&---------------------------------------------------------------------*
*&      Form  F_PENDECIA_APRODUZIR
*&---------------------------------------------------------------------*
FORM f_pendecia_aproduzir .

  LOOP AT t_saida.
    v_tabix = sy-tabix.

    CLEAR t_zrnppt_prev.
    READ TABLE t_zrnppt_prev WITH KEY matnr     = t_saida-matnr
                                      codigo    = t_saida-codigo
***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Início
                                      kunnr     = t_saida-cod_cli
***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Início
                                      werks     = t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Início
*    t_saida-pendencia = t_saida-prev_org - ( t_saida-ov + t_saida-op + t_saida-expedicao + t_saida-faturado ).
***Inserção - Hermiro Carvalho - PRJ01313 - 28/01/2016 - Início
    IF t_saida-matnr(2) = '54'.
      t_saida-pendencia = t_saida-prev_org - ( t_saida-ov + t_saida-op + t_saida-expedicao + t_saida-expedi_cen + t_saida-exp_inter + t_saida-faturado ).
    ELSE.
      t_saida-pendencia = t_saida-prev_org - ( t_saida-ov + t_saida-op + t_saida-expedicao + t_saida-expedi_cen + t_saida-faturado ).
    ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 28/01/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Fim
    IF t_saida-pendencia LE 0.
      t_saida-aproduzir = t_saida-op + t_saida-ov.
    ELSE.
      t_saida-aproduzir = t_saida-pendencia + t_saida-op + t_saida-ov.
      IF t_zrnppt_prev-peso_peca IS NOT INITIAL.
*        t_saida-pendencia_pec = ( t_saida-pendencia * 1000 ) / t_zrnppt_prev-peso_peca.
***Inserção - Hermiro Carvalho - PRJ01313 - 06/11/2015 - Início
*        t_saida-pendencia_pec =  t_saida-pendencia / t_zrnppt_prev-peso_peca.
        t_saida-pendencia_pec =  t_saida-pendencia / ( t_zrnppt_prev-peso_peca / 1000 ).
***Inserção - Hermiro Carvalho - PRJ01313 - 06/11/2015 - Fim
      ENDIF.
    ENDIF.

    IF t_saida-aproduzir IS NOT INITIAL AND t_zrnppt_prev-peso_peca IS NOT INITIAL.
*      t_saida-aproduzir_pec = ( t_saida-aproduzir * 1000 ) / t_zrnppt_prev-peso_peca.
***Inserção - Hermiro Carvalho - PRJ01313 - 06/11/2015 - Início
*      t_saida-aproduzir_pec =  t_saida-aproduzir / t_zrnppt_prev-peso_peca.
      t_saida-aproduzir_pec =  t_saida-aproduzir / ( t_zrnppt_prev-peso_peca / 1000 ).
***Inserção - Hermiro Carvalho - PRJ01313 - 06/11/2015 - Fim
    ENDIF.

* Calcula Necessidade de MP. Sandra H. - set/2010
    PERFORM f_necessidade_mp.

    IF t_saida-tamanho_lote IS NOT INITIAL AND t_saida-roteiro CA 'K'.
      t_saida-qtd_rack = t_saida-pendencia_pec / t_saida-tamanho_lote.
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 196432 - 05/04/2018 - Início
    CLEAR: v_instid.
    WRITE icon_annotation  AS ICON TO t_saida-call_obs.
***Inserção - Hermiro Carvalho - RDM 196432 - 05/04/2018 - Fim

    MODIFY t_saida INDEX v_tabix.
    CLEAR t_saida.
  ENDLOOP.

ENDFORM.                    " F_PENDECIA_APRODUZIR

*&---------------------------------------------------------------------*
*&      Form  F_IMPRIME_SAIDA
*&---------------------------------------------------------------------*
FORM f_imprime_saida .
****Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Início
*  DATA: wa_mslb     TYPE mslb,
*        wa_mslb_aux TYPE mslb.
****Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Fim
*
**PP - Cenário Estoque Consignado - RDM 239896
*  DATA: wa_consignado TYPE ty_consignado.
**PP - Cenário Estoque Consignado - RDM 239896
*
****Inserção - Hermiro Carvalho - 18/04/2016 - Início
*  DATA: v_095 TYPE p DECIMALS 3 VALUE '0.095',
*        v_005 TYPE p DECIMALS 3 VALUE '0.05',
****Inserção - Hermiro Carvalho - 18/04/2016 - Fim
*        vl_ceex TYPE p DECIMALS 3,
*        vl_cfat TYPE p DECIMALS 3,
*        vl_cpdc TYPE p DECIMALS 3,
*        vl_creq TYPE p DECIMALS 3,
*        vl_ctot TYPE p DECIMALS 3.
*  DATA: wa_pl_corte   TYPE zrnppt_pl_corte,
*        wa_lfa1      TYPE lfa1.
*  DATA: vl_log_mslb(80) TYPE c.
*
** >>> - 03.10.2019 14:06:46 - RDM 225838 - Simone
*  DATA: l_period TYPE sy-datum.
** <<< - 03.10.2019 14:06:46 - RDM 225838 - Simone
*
*  CLEAR: wa_mslb, t_mslb_aux[], t_mslb[].
*
*  DATA: wa_color TYPE lvc_s_scol.
*  "vl_index TYPE sy-tabix.
****Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*  DATA: vl_op    TYPE zrnppt_prev-peso.
****Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
*
**PP - Cenário Estoque Consignado - RDM 239896
*  DATA lt_werks_consignado TYPE STANDARD TABLE OF zsuget_variaveis WITH HEADER LINE.
**PP - Cenário Estoque Consignado - RDM 239896

  PERFORM f_define_layout.
  PERFORM f_monta_campos.

*GVCA20221020+ #4693-Ajuste na transação ZPP_PREVP (PRJ1353) Begin
  PERFORM f_monta_saida.
*GVCA20221020+ #4693-Ajuste na transação ZPP_PREVP (PRJ1353) End

***LOGICA MOVIDA PARA O FORM f_monta_saida****
****Inserção - Hermiro Carvalho - RDM 160097 - 12/04/2016 - Início
****Comentar - Hermiro Carvalho - RDM 160097 - 05/05/2016 - Início
*  IF t_saida[] IS NOT INITIAL.
**    CLEAR: r_ptnumber[], wa_ptnumber.
**    LOOP AT t_saida INTO wa_saida.
**      wa_ptnumber-sign   = 'I'.   "I = include, E = exclude
**      wa_ptnumber-option = 'EQ'.  "EQ, BT, NE ....
**      wa_ptnumber-low    = wa_saida-codigo.
**      CONDENSE wa_ptnumber-low NO-GAPS.
**      APPEND wa_ptnumber TO r_ptnumber.
**      CLEAR: wa_saida, wa_ptnumber.
**    ENDLOOP.
*
*    SELECT *
*      INTO TABLE t_mslb
*      FROM mslb
*      FOR ALL ENTRIES IN t_saida
*     WHERE matnr = t_saida-materia_prima
*       AND werks = t_saida-werks
*       AND lblab > 0.
*
*    LOOP AT t_mslb INTO wa_mslb.
*
*      vl_index = sy-tabix.
*
*      CLEAR vl_log_mslb.
*
*      PERFORM check_reserva_estoque_especial  USING wa_mslb-matnr
*                                                    wa_mslb-werks
*                                                    wa_mslb-charg
*                                              CHANGING vl_log_mslb.
*      IF vl_log_mslb IS NOT INITIAL.
*        DELETE t_mslb INDEX vl_index.
*      ENDIF.
*
*    ENDLOOP.
*
**PP - Cenário Estoque Consignado - RDM 239896
*    REFRESH: lt_werks_consignado,
*             t_consignado.
*    SELECT *
*      FROM zsuget_variaveis
*      INTO TABLE lt_werks_consignado
*     WHERE repid  = 'ZSUMMI_EDINFE'
*       AND field  = 'CONSIGNADOS'.
*    IF lt_werks_consignado[] IS NOT INITIAL.
*
*      CLEAR: r_werks_consignado[], r_werks_consignado.
*      LOOP AT lt_werks_consignado.
*        r_werks_consignado-sign   = 'I'.
*        r_werks_consignado-option = 'EQ'.
*        r_werks_consignado-low    = lt_werks_consignado-filtro.
*        APPEND r_werks_consignado.
*        CLEAR r_werks_consignado.
*      ENDLOOP.
*
*      SELECT matnr werks lgort charg ersda clabs
*        INTO TABLE t_consignado
*        FROM mchb
*        FOR ALL ENTRIES IN t_saida
*       WHERE matnr = t_saida-materia_prima
*         AND werks IN r_werks_consignado
*         AND clabs > 0.
*    ENDIF.
**PP - Cenário Estoque Consignado - RDM 239896
*
****Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Início
*    CLEAR: t_hist_lt[], t_mslb_ltan[], wa_mslb.
*    SELECT werks matnr idnrk
*      INTO TABLE t_hist_lt
*      FROM zsuppt_hist_lt
*       FOR ALL ENTRIES IN t_saida
*     WHERE werks EQ t_saida-werks
*      AND matnr  EQ t_saida-matnr.
*
*    IF t_hist_lt[] IS NOT INITIAL.
*      SELECT *
*        INTO TABLE t_mslb_ltan
*        FROM mslb
*        FOR ALL ENTRIES IN t_hist_lt
*       WHERE matnr = t_hist_lt-idnrk
*         AND werks = t_hist_lt-werks
*         AND lblab > 0.
*
*
*      LOOP AT t_mslb_ltan INTO wa_mslb.
*        vl_index = sy-tabix.
*        CLEAR vl_log_mslb.
*        PERFORM check_reserva_estoque_especial  USING wa_mslb-matnr
*                                                      wa_mslb-werks
*                                                      wa_mslb-charg
*                                                CHANGING vl_log_mslb.
*        IF vl_log_mslb IS NOT INITIAL.
*          DELETE t_mslb_ltan INDEX vl_index.
*        ENDIF.
*      ENDLOOP.
*
*    ENDIF.
****Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Fim
*
** >>> - 03.10.2019 14:06:46 - RDM 225838 - Simone
*    CLEAR: l_period, t_estoqtransito, t_estoqtransito[].
*    l_period = p_period-low - 30.
*    IF p_matpri-low IS INITIAL.
*      SELECT mblnr mjahr bwart matnr werks lgort charg menge budat_mkpf
*             INTO TABLE t_estoqtransito
*             FROM mseg
*             FOR ALL ENTRIES IN t_saida
*             WHERE   ( matnr = t_saida-matnr OR
*                       matnr = t_saida-materia_mp )     AND
*                     werks = t_saida-werks              AND
**  Início alteração LCS - RDM249282 - 09/11/2020
**                   ( bwart EQ '861' OR bwart EQ '862' ) AND
*                   ( bwart EQ '861' OR bwart EQ '862' OR bwart EQ '641' ) AND
**  Fim alteração LCS - RDM249282 - 09/11/2020
*                     budat_mkpf BETWEEN l_period AND p_period-high.
*    ELSE.
*      SELECT mblnr mjahr bwart matnr werks lgort charg menge budat_mkpf
*             INTO TABLE t_estoqtransito
*             FROM mseg
*             FOR ALL ENTRIES IN t_saida
*             WHERE   matnr = t_saida-materia_mp         AND
*                     werks = t_saida-werks              AND
**  Início alteração LCS - RDM249282 - 09/11/2020
**                   ( bwart EQ '861' OR bwart EQ '862' ) AND
*                   ( bwart EQ '861' OR bwart EQ '862' OR bwart EQ '641' ) AND
**  Fim alteração LCS - RDM249282 - 09/11/2020
*                     budat_mkpf BETWEEN l_period AND p_period-high.
*    ENDIF.
*
*    IF sy-subrc EQ 0.
*      SORT t_estoqtransito BY matnr bwart.
*    ENDIF.
** <<< - 03.10.2019 14:06:46 - RDM 225838 - Simone
*  ENDIF.
****Comentar - Hermiro Carvalho - RDM 160097 - 05/05/2016 - Fim
****Inserção - Hermiro Carvalho - RDM 160097 - 12/04/2016 - Fim
*
*  PERFORM f_busca_ressup_clge.
*
*
*****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
*  CLEAR: wa_saida, vl_index, vl_op.
*  LOOP AT t_saida INTO wa_saida.
*    vl_index = sy-tabix.
*
*    IF p_inati IS NOT INITIAL."Verifica parâmetro de tela.
*      CLEAR: vl_op.
*      vl_op = wa_saida-op_lib + wa_saida-op_aber.
*      IF vl_op > '0.000' OR wa_saida-expedicao > '0.000'."Verifica condição
*        DELETE t_saida INDEX vl_index.
*        CONTINUE.
*      ENDIF."Verifica condição
*    ENDIF."Verifica parâmetro de tela.
*    CASE wa_saida-nivel_est.
*      WHEN 'SEM MP'.
*        wa_color-fname     = 'NIVEL_EST'.
*        wa_color-color-col = '6'."Vermelho
*        APPEND wa_color TO wa_saida-cellcolor.
*        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
*        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*
*      WHEN 'SEM MP C/ MIN'.
*        wa_color-fname     = 'NIVEL_EST'.
*        wa_color-color-col = '7'."Laranja
*        APPEND wa_color TO wa_saida-cellcolor.
*        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
*        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*
*      WHEN 'SEM OP'.
*        wa_color-fname     = 'NIVEL_EST'.
*        wa_color-color-col = '1'."Seria Lilás, mas o sab não tem essa cor
*        APPEND wa_color TO wa_saida-cellcolor.
*        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
*        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*
*      WHEN 'A PRODUZIR'.
*        wa_color-fname     = 'NIVEL_EST'.
*        wa_color-color-col = '3'."Amarelo
*        APPEND wa_color TO wa_saida-cellcolor.
*        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
*        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*
*      WHEN 'MÍNIMO'.
*        wa_color-fname     = 'NIVEL_EST'.
*        wa_color-color-col = '5'."Amarelo
*        APPEND wa_color TO wa_saida-cellcolor.
*        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
*        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*
*    ENDCASE.
****Inserção - Hermiro Carvalho - RDM 160097 - 12/04/2016 - Início
*
*** Estoque no Fornecedor (T_SAIDA-ESTOQUE_FORN)
*    LOOP AT t_mslb INTO wa_mslb WHERE matnr = wa_saida-materia_prima AND
*                                      werks = wa_saida-werks.
*      wa_saida-estoque_forn = wa_mslb-lblab + wa_saida-estoque_forn.
*    ENDLOOP.
*
**PP - Cenário Estoque Consignado - RDM 239896
*** Estoque Consignado (T_SAIDA-ESTOQUE_CONSIGNADO)
*    LOOP AT t_consignado INTO wa_consignado WHERE matnr = wa_saida-materia_prima.
*      wa_saida-estoque_consignado = wa_consignado-clabs + wa_saida-estoque_consignado.
*    ENDLOOP.
**PP - Cenário Estoque Consignado - RDM 239896
*
****Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Início
*** Estoque no Fornecedor de LT Antiga (T_SAIDA-ESTOQUE_FORN_LTAN)
*    LOOP AT t_hist_lt INTO wa_hist_lt WHERE mantr = wa_saida-matnr AND
*                                            werks = wa_saida-werks.
*      LOOP AT t_mslb_ltan INTO wa_mslb WHERE matnr = wa_hist_lt-idnrk AND
*                                             werks = wa_hist_lt-werks.
*        wa_saida-estoque_forn_ltan = wa_mslb-lblab + wa_saida-estoque_forn_ltan.
*      ENDLOOP.
*      CLEAR: wa_hist_lt, wa_mslb.
*    ENDLOOP.
****Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Fim
*
****Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Início
**    t_mslb_aux[] = t_mslb[].
**    DELETE t_mslb_aux WHERE matnr <> wa_saida-matnr
**                        AND werks <> wa_saida-werks.
**    IF t_ekko IS NOT INITIAL.
**      CLEAR: wa_ekko.
**      SORT t_ekko BY matnr werks.
**      READ TABLE t_ekko INTO wa_ekko WITH KEY matnr = wa_saida-matnr
**                                              werks = wa_saida-werks.
**      CASE sy-subrc."Verifica se encontrou registro na t_ekko - Início
**        WHEN 0."Encontrou
**          READ TABLE t_mslb_aux INTO wa_mslb_aux WITH KEY matnr = wa_ekko-matnr
**                                                          werks = wa_ekko-werks
**                                                          lifnr = wa_ekko-lifnr.
**          CASE sy-subrc."Verifica se encontrou registro na t_mslb_aux - Início
**            WHEN 0."Encontrou
**              IF wa_mslb_aux-mp_forn GT 0.
**                wa_saida-mp_forn = wa_mslb_aux-mp_forn.
**              ENDIF.
**            WHEN OTHERS."Não encontrou
**          ENDCASE."Verifica se encontrou registro na t_mslb_aux - Fim
**        WHEN OTHERS."Não encontrou
**      ENDCASE."Verifica se encontrou registro na t_ekko - Fim
**    ENDIF.
**    SORT: t_mslb_aux BY dt_atualiza DESCENDING.
**    LOOP AT t_mslb_aux INTO wa_mslb_aux."Verifica as datas - Início
**      CASE wa_mslb-dt_atualiza."Campo data estrutura auxiliar - Início
**        WHEN space."Quando for vazio
**          wa_mslb = wa_mslb_aux.
**        WHEN OTHERS."Quando estiver preenchido.
**          IF wa_mslb_aux-dt_atualiza < wa_mslb-dt_atualiza.
**            DELETE t_mslb_aux FROM wa_mslb_aux.
**          ENDIF.
**          IF wa_mslb_aux-dt_atualiza > wa_mslb-dt_atualiza.
**            wa_mslb = wa_mslb_aux.
**          ENDIF.
**          CONTINUE."Continua para o próximo registro
**      ENDCASE."Campo data estrutura auxiliar - Fim
**    ENDLOOP."Verifica as datas - Fim
**    wa_saida-expedicao_forn = wa_mslb-expedicao_forn.
**    wa_saida-mp_fis_forn    = wa_mslb-mp_forn.
**    wa_saida-processo_forn  = wa_mslb-processo_forn.
****Comentar - Hermiro Carvalho - RDM 160097 - 16/05/2016 - Início
*
*** Estoque + Faturado (T_SAIDA-ESTFAT)
****Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Início
**    wa_saida-estfat         = wa_saida-expedicao + wa_saida-expedicao_forn +
**                              wa_saida-faturado1 + wa_saida-faturado1_cen.
*    wa_saida-estfat         = wa_saida-expedicao + wa_saida-expedi_cen +
*                              wa_saida-expedicao_forn + wa_saida-faturado1 +
*                              wa_saida-faturado1_cen.
****Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Fim
****Comentar - Hermiro Carvalho - RDM 160097 - 16/05/2016 - Fim
****Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Fim
*
*
***Volume a produzir
*    "wa_saida-volume_aproduzir = (t_saida-aproduzir)   (confirmar substituição da regra atual)
****Comentar - Hermiro Carvalho - RDM 160097 - 18/04/2016 - Início
**    wa_saida-vol_aproduzir = 1.
**    IF ( v_095 * wa_saida-prev_org ) > wa_saida-estfat.
***    wa_saida-volume_aproduzir = wa_saida-prev_org - "coluna Estoque + Faturado" + wa_saida-estoque_minimo.
**      wa_saida-vol_aproduzir = wa_saida-prev_org + wa_saida-estoque_minimo.
**    ELSE.
**      IF ( wa_saida-prev_org + wa_saida-expedicao_forn ) > wa_saida-estoque_minimo.
**        wa_saida-vol_aproduzir = 0.
**      ELSE.
**        wa_saida-vol_aproduzir = wa_saida-estoque_minimo.
**      ENDIF.
**    ENDIF.
****Comentar - Hermiro Carvalho - RDM 160097 - 18/04/2016 - Fim
*
**wa_saida-vol_aproduzir = wa_saida-aproduzir.    "Não está com a nova regra que foi desenhada no chamado RDM 160097
*
** RDM 216498 RGC8096 04Abr2019
*    wa_saida-vol_aproduzir = wa_saida-prev_org - wa_saida-estfat.
** RDM 216498 RGC8096 04Abr2019
*
***Volume a programar
*
**    IF ( v_095 * ( wa_saida-vol_aproduzir - wa_saida-requisicoes - wa_saida-pedidos ) < v_005 ).
*****Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Início
***      wa_saida-vol_aproduzir = 0.
**      wa_saida-vol_aprogramar = 0.
*****Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Fim
**    ELSE.
**      IF ( ( wa_saida-vol_aproduzir - wa_saida-requisicoes - wa_saida-pedidos ) < wa_saida-estoque_minimo ).
**        wa_saida-vol_aprogramar = wa_saida-estoque_minimo.
**      ELSE.
**        wa_saida-vol_aprogramar = wa_saida-vol_aproduzir - wa_saida-requisicoes - wa_saida-pedidos.
**      ENDIF.
**    ENDIF.
*
** RDM 216498 RGC8096 04Abr2019
*    wa_saida-vol_aprogramar = wa_saida-prev_org -
*    ( wa_saida-estfat + wa_saida-op_lib +
*      wa_saida-op_aber + wa_saida-op_lib_cen +
*        wa_saida-op_aber_cen ).
** RDM 216498 RGC8096 04Abr2019
*
***Estoque 1 dia
**      wa_saida-estoque1dia = wa_saida-estoque_minimo / (parâmetro da nova tabela de estoque mínimo).
****Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Início
**    IF wa_saida-estoque_minimo > 0.
**      wa_saida-estoque1dia = wa_saida-estoque_minimo / 1.
**    ENDIF.
**    SELECT SINGLE matnr werks rwpro
**      INTO wa_marc
**      FROM marc
**     WHERE matnr = wa_saida-matnr
**       AND werks = wa_saida-werks.
**
**    SELECT SINGLE rwpro mirwx
**      INTO wa_t438u
**      FROM t438u
**     WHERE rwpro EQ wa_marc-rwpro.
**    IF wa_t438u-mirwx > 0.
***      wa_saida-estoque1dia = wa_saida-estoque_minimo / wa_t438u-mirwx.
**      wa_saida-est_min_dias = wa_t438u-mirwx.
**      wa_saida-estoque1dia = wa_saida-estoque_minimo / wa_t438u-mirwx.
**    ENDIF.
****Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Fim
*****Inserção - Sandra H. - RDM 174627 - 18/01/2017 - Início
*    IF wa_saida-est_min_dias > 0.
*      wa_saida-estoque1dia = wa_saida-estoque_minimo / wa_saida-est_min_dias.
*    ENDIF.
*****Inserção - Sandra H. - RDM 174627 - 18/01/2017 - FIM
*
*** Processo Fornecedor (T_SAIDA-PROCESSO_FORN)
*** <pendente a criação da tabela por parte do modulo de MM>
*
***Cobertura Estoque Expedição
*    CASE wa_saida-prev_org.
*      WHEN 0.
*        wa_saida-ceex      = 'SEM DEMANDA'.
****Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Início
*        wa_color-fname     = 'CEEX'.
*        wa_color-color-col = '5'."Verde
*        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
****Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Fim
*      WHEN OTHERS.
*        IF wa_saida-estoque1dia > 0.
*          CLEAR vl_ceex.
*          vl_ceex = ( wa_saida-expedicao + wa_saida-expedicao_forn ) / wa_saida-estoque1dia.
*          wa_saida-ceex = vl_ceex.
*          CONDENSE wa_saida-ceex NO-GAPS.
*          TRANSLATE wa_saida-ceex USING  '.,'.
****Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Início
*          IF vl_ceex > 30.
*            wa_color-fname     = 'CEEX'.
*            wa_color-color-col = '5'."Verde
*            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*          ENDIF.
*          IF vl_ceex BETWEEN 15 AND 30.
*            wa_color-fname     = 'CEEX'.
*            wa_color-color-col = '3'."Amarelo
*            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*
*          ENDIF.
*          IF vl_ceex < 15.
*            wa_color-fname     = 'CEEX'.
*            wa_color-color-col = '7'."Laranja
*            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*          ENDIF.
*          IF vl_ceex = 0.
*            wa_color-fname     = 'CEEX'.
*            wa_color-color-col = '6'."Vermelho
*            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*          ENDIF.
*          IF wa_saida-estoque_minimo = 0.
*            wa_color-fname     = 'CEEX'.
*            wa_color-color-col = '1'."Vermelho
*            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
*          ENDIF.
****Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Fim
*        ELSE.
*          IF wa_saida-vol_aproduzir > 0.
*            wa_saida-ceex = 'PRODUZIR SÓ DEMANDA'.
*          ELSE.
*            wa_saida-ceex = 'SEM DEMANDA'.
****Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Início
*            wa_color-fname     = 'CEEX'.
*            wa_color-color-col = '5'."Verde
*            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
****Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Fim
*          ENDIF.
*        ENDIF.
*    ENDCASE.
****Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Início
*    APPEND wa_color TO wa_saida-cellcolor.
*    MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
****Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Fim
*
***Cobertura Faturado
*    IF wa_saida-estoque1dia > 0.
*      CLEAR vl_cfat.
*      vl_cfat       = ( wa_saida-faturado1 + wa_saida-faturado1_cen ) / wa_saida-estoque1dia.
*      wa_saida-cfat = vl_cfat.
*      CONDENSE wa_saida-cfat NO-GAPS.
*      TRANSLATE wa_saida-cfat USING  '.,'.
*    ELSE.
*      IF wa_saida-vol_aproduzir > 0.
*        wa_saida-cfat = 'PRODUZIR SÓ DEMANDA'.
*      ELSE.
*        wa_saida-cfat = 'SEM DEMANDA'.
*      ENDIF.
*    ENDIF.
*
***Cobertura Pedidos Compras
*    IF wa_saida-estoque1dia > 0.
*      CLEAR vl_cpdc.
*      vl_cpdc = ( wa_saida-pedidos - wa_saida-expedicao_forn ) / wa_saida-estoque1dia.
*      wa_saida-cpdc = vl_cpdc.
*      CONDENSE wa_saida-cpdc NO-GAPS.
*      TRANSLATE wa_saida-cpdc USING  '.,'.
*    ELSE.
*      IF wa_saida-vol_aproduzir > 0.
*        wa_saida-cpdc = 'PRODUZIR SÓ DEMANDA'.
*      ELSE.
*        wa_saida-cpdc = 'SEM DEMANDA'.
*      ENDIF.
*    ENDIF.
*
*
***Cobertura Requisições
*    IF wa_saida-estoque1dia > 0.
*      CLEAR vl_creq.
*      vl_creq = wa_saida-requisicoes / wa_saida-estoque1dia.
*      wa_saida-creq = vl_creq.
*      CONDENSE wa_saida-creq NO-GAPS.
*      TRANSLATE wa_saida-creq USING  '.,'.
*    ELSE.
*      IF wa_saida-vol_aproduzir > 0.
*        wa_saida-creq = 'PRODUZIR SÓ DEMANDA'.
*      ELSE.
*        wa_saida-creq = 'SEM DEMANDA'.
*      ENDIF.
*    ENDIF.
*
*
***Cobertura Total
*    IF wa_saida-creq = 'PRODUZIR SÓ DEMANDA'.
*      wa_saida-ctot = 'PRODUZIR SÓ DEMANDA'.
*    ELSE.
*      IF wa_saida-creq = 'SEM DEMANDA'.
*        wa_saida-ctot = 'SEM DEMANDA'.
*      ELSE.
*        CLEAR vl_ctot.
*        vl_ctot = vl_ceex + vl_cfat + vl_cpdc + vl_creq.
*        wa_saida-ctot = vl_ctot.
*        CONDENSE wa_saida-ctot NO-GAPS.
*        TRANSLATE wa_saida-ctot USING  '.,'.
*      ENDIF.
*    ENDIF.
*
*
***Status Cobertura Estoque Expedição
*    IF wa_saida-creq = 'PRODUZIR SÓ DEMANDA'.
*      wa_saida-scee = 'PRODUZIR SÓ DEMANDA'.
*    ENDIF.
*
*    IF wa_saida-scee IS INITIAL.
*      IF wa_saida-creq = 'SEM DEMANDA'.
*        wa_saida-scee = 'SEM DEMANDA'.
*      ENDIF.
*    ENDIF.
*
*    IF wa_saida-scee IS INITIAL.
*      IF vl_ceex < 7.
*        wa_saida-scee = 'SEM ESTOQUE'.
*      ENDIF.
*    ENDIF.
*
*    IF wa_saida-scee IS INITIAL.
*      IF vl_ceex < 15.
*        wa_saida-scee = 'MENOR DE 15 DIAS'.
*      ENDIF.
*    ENDIF.
*
*    IF wa_saida-scee IS INITIAL.
*      IF vl_ceex < 30.
*        wa_saida-scee = 'ENTRE 15 e 30 DIAS'.
*      ELSE.
*        wa_saida-scee = 'ACIMA DE 30 DIAS'.
*      ENDIF.
*    ENDIF.
****Inserção - Hermiro Carvalho - RDM 172372 - 01/12/2016 - Início
**    IF t_saida-materia_prima <> '*'.
*    IF wa_saida-materia_prima <> '*'.
*      PERFORM f_texto_material USING    wa_saida-materia_prima
*                               CHANGING wa_saida-desc_mp.
*    ENDIF.
****Inserção - Hermiro Carvalho - RDM 172372 - 01/12/2016 - Fim
****Inserção - Hermiro Carvalho - RDM 184185 - 18/09/2017 - Início
**    CLEAR: wa_pl_corte, wa_lfa1.
**    SELECT SINGLE *
**      INTO wa_pl_corte
**      FROM zrnppt_pl_corte
**     WHERE num_pl_corte = wa_saida-num_pl_corte.
**    IF sy-subrc IS INITIAL.
**      SELECT SINGLE *
**        INTO wa_lfa1
**        FROM lfa1
**       WHERE lifnr =  wa_pl_corte-lifnr.
**      IF sy-subrc IS INITIAL.
**        t_saida-sortl = wa_lfa1-sortl.
**      ENDIF.
**    ENDIF.
****Inserção - Hermiro Carvalho - RDM 184185 - 18/09/2017 - Fim
****Inserção - Hermiro Carvalho - RDM 196432 - 19/04/2018 - Início
*    CLEAR: wa_saida-observacao.
**INC 2885055 - 31.07.19 - inicio
**    PERFORM verifica_primeira_linha USING p_centro-low wa_saida-matnr p_period-low CHANGING wa_saida-observacao.
*    PERFORM verifica_primeira_linha USING wa_saida-werks wa_saida-matnr p_period-low CHANGING wa_saida-observacao.
**INC 2885055 - 31.07.19 - fim
*
****Inserção - Hermiro Carvalho - RDM 196641 - 06/08/2018 - Início
****Rotina para preenchimento dos novos campos de cobertura
*    PERFORM calculo_cobertura USING wa_saida wa_color CHANGING wa_saida wa_saida-cellcolor.
****Inserção - Hermiro Carvalho - RDM 196641 - 06/08/2018 - Fim
****Inserção - Hermiro Carvalho - RDM 196432 - 19/04/2018 - Fim
*
*    READ TABLE it_vsr_clge INTO wa_vsr_clge WITH KEY centro  = wa_saida-werks
*                                                     cd_mp   = wa_saida-materia_prima.
*    IF sy-subrc EQ 0.
*
*      IF     wa_vsr_clge-peso_01 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_01.
*        wa_saida-data_entre = wa_vsr_clge-data_01.
*      ELSEIF wa_vsr_clge-peso_02 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_02.
*        wa_saida-data_entre = wa_vsr_clge-data_02.
*      ELSEIF wa_vsr_clge-peso_03 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_03.
*        wa_saida-data_entre = wa_vsr_clge-data_03.
*      ELSEIF wa_vsr_clge-peso_04 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_04.
*        wa_saida-data_entre = wa_vsr_clge-data_04.
*      ELSEIF wa_vsr_clge-peso_05 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_05.
*        wa_saida-data_entre = wa_vsr_clge-data_05.
*      ELSEIF wa_vsr_clge-peso_06 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_06.
*        wa_saida-data_entre = wa_vsr_clge-data_06.
*      ELSEIF wa_vsr_clge-peso_07 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_07.
*        wa_saida-data_entre = wa_vsr_clge-data_07.
*      ELSEIF wa_vsr_clge-peso_08 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_08.
*        wa_saida-data_entre = wa_vsr_clge-data_08.
*      ELSEIF wa_vsr_clge-peso_09 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_09.
*        wa_saida-data_entre = wa_vsr_clge-data_09.
*      ELSEIF wa_vsr_clge-peso_10 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_10.
*        wa_saida-data_entre = wa_vsr_clge-data_10.
*      ELSEIF wa_vsr_clge-peso_11 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_11.
*        wa_saida-data_entre = wa_vsr_clge-data_11.
*      ELSEIF wa_vsr_clge-peso_12 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_12.
*        wa_saida-data_entre = wa_vsr_clge-data_12.
*      ELSEIF wa_vsr_clge-peso_13 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_13.
*        wa_saida-data_entre = wa_vsr_clge-data_13.
*      ELSEIF wa_vsr_clge-peso_14 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_14.
*        wa_saida-data_entre = wa_vsr_clge-data_14.
*      ELSEIF wa_vsr_clge-peso_15 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_15.
*        wa_saida-data_entre = wa_vsr_clge-data_15.
*      ELSEIF wa_vsr_clge-peso_16 > 0.
*        wa_saida-qtd_mp_che = wa_vsr_clge-peso_16.
*        wa_saida-data_entre = wa_vsr_clge-data_16.
*      ENDIF.
*    ENDIF.
*
****Inicio - Fabio - 14.08.2020 - RDM 238614
*    PERFORM f_ler_divisao_remessa USING wa_saida-data_inicio
*                                        wa_saida-volume_rem
*                                        wa_saida-volume_rem_pc
*                                        wa_saida-volume_rem_1
*                                        wa_saida-volume_rem_1_pc.
*
*    WRITE '@B_TVAR@' CENTERED TO wa_saida-volume_icon.
****Fim - Fabio - 14.08.2020 - RDM 238614
*
*    wa_saida-line = vl_index.
*    MODIFY t_saida FROM wa_saida INDEX vl_index.
****Inserção - Hermiro Carvalho - RDM 160097 - 12/04/2016 - Fim
*
**RDM 238614
*    CLEAR wa_saida.
**RDM 238614
*
*  ENDLOOP.
*
** >>> - 03.10.2019 14:06:46 - RDM 225838 - Simone
*  CLEAR: wa_saida, vl_index.
*  LOOP AT t_saida INTO wa_saida.
*    vl_index = sy-tabix.
*    READ TABLE t_estoqtransito WITH KEY matnr = wa_saida-matnr.
*    IF sy-subrc EQ 0.
*      PERFORM pesquisa_estoque_transito USING    wa_saida-matnr
*                                        CHANGING wa_saida-estqtran.
*    ELSE.
*      PERFORM pesquisa_estoque_transito USING    wa_saida-materia_mp
*                                        CHANGING wa_saida-estqtran.
*    ENDIF.
*    MODIFY t_saida FROM wa_saida INDEX vl_index.
*  ENDLOOP.
** <<< - 03.10.2019 14:06:46 - RDM 225838 - Simone

****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim
***Inserção - Hermiro Carvalho - RDM 196162 - 20/06/2018 - Início
  IF sy-ucomm EQ 'ATUA'.
    EXIT.
  ENDIF.
***Inserção - Hermiro Carvalho - RDM 196162 - 20/06/2018 - Fim
***Comentar - Hermiro Carvalho - RDM 196641 - 09/08/2018 - Início
  IF op1 = 'X'.
    PERFORM ajusta_tabela_saida."Ajusta a tabela de saída para o layout sintético
  ENDIF.

*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896
  IF gv_program NE 'ZPPR_GATILHO_JIT_CONSIGNADO'.
*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896

* Exporta dados para extranet
* RGC8096 RDM 210489 27Nov2018

* Stefanini/Cristiane Souza – RDM 338761 - 06/11/2026 - Início
    IF sy-tcode NE 'ZPP_PREVP'.
* Stefanini/Cristiane Souza – RDM 338761 - 06/11/2026 - Fim
*    IF p_0001 IS INITIAL.                          "RDM324238(-)
      IF p_0001 IS INITIAL AND p_csv IS INITIAL AND sy-cprog NE 'ZRNPPR_PLANEJ_PROD_V2'. "RDM324238
        IF ( sy-tcode = 'ZPP_SAP_EXTRANET' OR sy-tcode = 'ZPP_SAP_EXTRANET_TAB' OR sy-tcode = 'SM37' OR sy-tcode = 'SE38' OR sy-batch IS NOT INITIAL ).
          EXPORT t_saida TO MEMORY ID 'EXTRANET'.
          LEAVE SCREEN.
        ENDIF.
      ENDIF.
* Stefanini/Cristiane Souza – RDM 338761 - 06/11/2026 - Início
    ENDIF.
* Stefanini/Cristiane Souza – RDM 338761 - 06/11/2026 - Fim

***Comentar - Hermiro Carvalho - RDM 196641 - 09/08/2018 - Fim
***Inserção - Hermiro Carvalho - RDM 196641 - 07/08/2018 - Início
***Chamada do ALV para as opções 'Analítico' e 'Sintético'

*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896
  ENDIF.
*PP - Cenário Estoque Consignado - gatilho JIT - RDM 239896

  DATA: l_ausch TYPE ausch.

  FIELD-SYMBOLS: <fssaida> LIKE LINE OF t_saida.

  LOOP AT t_saida ASSIGNING <fssaida>.
    CALL FUNCTION 'ZPPF_OBTEM_PERDA'
      EXPORTING
        werks   = <fssaida>-werks
        matnr   = <fssaida>-matnr
      IMPORTING
        e_ausch = l_ausch.
    <fssaida>-ausch = l_ausch.
  ENDLOOP.

  DATA : i_grid_settings TYPE lvc_s_glay.

*  IF p_0001 IS INITIAL.                          "RDM324238(-)
  IF p_0001 IS INITIAL AND p_csv IS INITIAL.                "RDM324238
    IF p_alv_2 IS INITIAL.
      IF op1 IS NOT INITIAL OR op2 IS NOT INITIAL.
        CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
          EXPORTING
            i_callback_program       = sy-repid
            i_grid_settings          = i_grid_settings
            is_layout                = gs_layout
            i_callback_pf_status_set = 'F_PF_STATUS_SET'
            i_callback_top_of_page   = 'F_TOP_OF_PAGE'
            i_callback_user_command  = 'F_USER_COMMAND'
            it_fieldcat              = gt_fieldcat[]
            it_sort                  = gt_sort[]
            i_default                = 'X'
            i_save                   = 'A'
            is_variant               = gs_variante
          TABLES
            t_outtab                 = t_saida
          EXCEPTIONS
            program_error            = 1
            OTHERS                   = 2.


        "for call back values from editable grid cells
        i_grid_settings-edt_cll_cb = 'X'.

      ENDIF.

    ELSE.

      CALL SCREEN 9000.
    ENDIF.
  ENDIF.

* PP - Interface SAP x GD - RDM 227749 RGC8096
  DATA : lv_fname      TYPE string.
  DATA : lv_file       TYPE string.

  IF p_0001 IS NOT INITIAL.
*    PERFORM color_hexa.

*    DATA : lv_fname      TYPE string.
*    DATA : lv_file       TYPE string.
*    DATA : lv_file_color TYPE string.

    lv_fname = p_file.

    LOOP AT gt_fieldcat.
      IF gt_fieldcat-icon = 'X'.
        CONTINUE.
      ENDIF.
      MOVE-CORRESPONDING gt_fieldcat TO field_definition.
      APPEND field_definition.
    ENDLOOP.

    LOOP AT t_saida.
      MOVE-CORRESPONDING t_saida TO t_final.
      APPEND t_final.
    ENDLOOP.

    IF p_locl IS NOT INITIAL.

      PERFORM save_local.

      CONCATENATE lv_fname(8) '\' 'dwsu_definition.txt' INTO lv_file.

      CALL FUNCTION 'GUI_DOWNLOAD'
        EXPORTING
          filename              = lv_file
          filetype              = 'ASC'
          write_lf              = 'X'
          write_field_separator = '|'
        TABLES
          data_tab              = field_definition     " Pass the Output internal table
        EXCEPTIONS
          OTHERS                = 22.

*      lv_file_color = p_file.
*      REPLACE ALL OCCURRENCES OF '.txt' IN lv_file_color WITH '_'.
*      CONDENSE lv_file_color.

**     CONCATENATE lv_fname(8) '\' 'dwsu_color.txt' INTO lv_file.
*      CONCATENATE lv_file_color 'color.txt' INTO lv_file.

*      CALL FUNCTION 'GUI_DOWNLOAD'
*        EXPORTING
*          filename              = lv_file
*          filetype              = 'ASC'
*          write_lf              = 'X'
*          write_field_separator = '|'
*        TABLES
*          data_tab              = t_color    " Pass the Output internal table
*        EXCEPTIONS
*          OTHERS                = 22.
    ELSE.

      PERFORM save_server.

    ENDIF.
*RDM324238 - Begin
  ELSEIF p_csv IS NOT INITIAL.
    lv_fname = p_file.

    LOOP AT gt_fieldcat.
      IF gt_fieldcat-icon = 'X'.
        CONTINUE.
      ENDIF.
      MOVE-CORRESPONDING gt_fieldcat TO field_definition.
      APPEND field_definition.
    ENDLOOP.

    LOOP AT t_saida.
      MOVE-CORRESPONDING t_saida TO t_final.
      APPEND t_final.
    ENDLOOP.

    IF p_locl IS NOT INITIAL.
      PERFORM save_local.

*      CONCATENATE lv_fname(8) '\' 'EXPORT.csv' INTO lv_file.
*
*      CALL FUNCTION 'GUI_DOWNLOAD'
*        EXPORTING
*          filename              = lv_file
*          filetype              = 'ASC'
*          write_lf              = 'X'
*          write_field_separator = '|'
*        TABLES
*          data_tab              = field_definition     " Pass the Output internal table
*        EXCEPTIONS
*          OTHERS                = 22.
    ELSE.
      PERFORM save_server.
    ENDIF.
*RDM324238 - End
  ENDIF.
* PP - Interface SAP x GD - RDM 227749 RGC8096

  IF op3 IS NOT INITIAL.
***Tratamento dos dados de saída para a opção 'Resumo'
    PERFORM trata_dados_opcao_resumo.
  ENDIF.
***Inserção - Hermiro Carvalho - RDM 196641 - 07/08/2018 - Fim

ENDFORM.                    " F_IMPRIME_SAIDA

*&--------------------------------------------------------------------*
*&      Form  f_pf_status_set
*&--------------------------------------------------------------------*
*       Permite incluir ou excluir os botões do ALV
*---------------------------------------------------------------------*
FORM f_pf_status_set USING rt_extab TYPE slis_t_extab.

  REFRESH rt_extab.
  SET PF-STATUS 'STANDARD' EXCLUDING rt_extab.

ENDFORM.                    "f_pf_status_set

*&---------------------------------------------------------------------*
*&      Form  F_DEFINE_LAYOUT
*&---------------------------------------------------------------------*
FORM f_define_layout .

  gs_layout-group_change_edit    = 'X'.
  gs_layout-box_tabname          = 'T_SAIDA'.   "Tabela de saída
  gs_layout-zebra                = 'X'.         "Listrado (Linha clara->escura)
  gs_layout-colwidth_optimize    = 'X'.
  gs_layout-box_fieldname        = 'MARK'.
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Inicio
  gs_layout-coltab_fieldname     = 'CELLCOLOR'.
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim

  ls_layo-edit_mode  = 'X'.
  ls_layo-zebra      = 'X'.         "Listrado (Linha clara->escura)
  ls_layo-cwidth_opt = 'X'.
  ls_layo-box_fname    = 'MARK'.
  ls_layo-ctab_fname   = 'CELLCOLOR'.
  ls_layo-sel_mode     = 'A'.

ENDFORM.                    " F_DEFINE_LAYOUT

*&---------------------------------------------------------------------*
*&      Form  F_MONTA_CAMPOS
*&---------------------------------------------------------------------*
FORM f_monta_campos .

  "PERFORM f_monta_quebra.


  IF op1 = 'X'.
    SORT t_saida BY cliente.
    PERFORM:
***Inserção - Hermiro Carvalho - RDM 196641 - 07/08/2018 - Início
    f_inclui_campo USING 'T_SAIDA' 'WERKS'            'CENTRO'                 'C' '18' 'X' ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'MATNR'            'MATERIAL'               'C' '40' 'X' ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'LGORT'            'DEPÓSITO'               'C' '4' 'X'  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'CLIENTE'          'CLIENTE'                'C' '18' 'X' ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'COD_CLI'          'CÓDIGO CLIENTE'         'C' '18' ''  ''  '' '',
***Inicio - Fabio - 26.01.2021 - RDM 251520
    f_inclui_campo USING 'T_SAIDA' 'TP_CLIENTE'       'TIPO CLIENTE'           'C' '18' 'X'  ''  '' ''.
***Inicio - Fabio - 26.01.2021 - RDM 251520
    PERFORM:   f_inclui_campo USING 'T_SAIDA' 'COBFAT_EXP_ORG'     'COBERTURA Fat+Expedição ORG.' 'C' '20' ''  ''  '' '',
               f_inclui_campo USING 'T_SAIDA' 'COB_EXP_ORG'        'COBERTURA EXPEDIÇÃO ORG.'     'C' '20' ''  ''  '' '',
               f_inclui_campo USING 'T_SAIDA' 'COB_FAT_ORG'        'COBERTURA FATURADO ORG.'      'C' '20' ''  ''  '' '',
               f_inclui_campo USING 'T_SAIDA' 'COB_OP_TOTAL_DEM_ORG' 'COBERTURA OP TOTAL DEMANDA ORG.' 'C' '20' ''  ''  '' '',
               f_inclui_campo USING 'T_SAIDA' 'COB_MP_FORNE_DEM_ORG' 'COBERTURA MP EM FORNECEDOR  DEMANDA ORG.' 'C' '20' ''  ''  '' '',
               f_inclui_campo USING 'T_SAIDA' 'COB_PED_COMP_ORG'     'COBERTURA DE PEDIDOS COMPRA ORG.' 'C' '20' ''  ''  '' '',
               f_inclui_campo USING 'T_SAIDA' 'COB_EXP_60'           'COBERTURA EXPEDIÇÃO (60 DIAS)' 'C' '20' ''  ''  '' '',
               f_inclui_campo USING 'T_SAIDA' 'COB_STAT_EXP_60'      'STATUS COBERTURA Expedição (60 DIAS)' 'C' '20' ''  ''  '' '',
               f_inclui_campo USING 'T_SAIDA' 'S_MP'                 'S/MP'                                 'C' '6' ''  ''  'X' ''.
***Inserção - Hermiro Carvalho - RDM 196641 - 07/08/2018 - Fim
* >>> - 03.10.2019 14:06:46 - RDM 225838 - Simone
* <<< - 03.10.2019 14:06:46 - RDM 225838 - Simone

    PERFORM f_inclui_campo USING 'ZRNPPT_PREV' 'STATUS_PCP' 'STATUS_PCP' 'C' '60' '' 'X' '' 'X'.
*    PERFORM f_inclui_campo USING 'ZRNPPT_PREV' 'STATUS_OM'  'STATUS_OM'  '' '60' '' 'X' '' 'X'.

    PERFORM f_inclui_campo USING 'T_SAIDA' 'QTD_MP_CHE' 'MP USINA CHEGAR'  'P' '18' '' 'X' 'X' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'DATA_ENTRE' 'DATA ENTR. USINA'  'D' '10' '' ''  ''  ''.



  ELSEIF op2 = 'X'.
    PERFORM:
    f_inclui_campo USING 'T_SAIDA' 'MATNR'            'MATERIAL'               'C' '18' 'X' '' 'X' '',
***Inserção - Hermiro Carvalho - RDM 196342 - 17/04/2018 - Início
    f_inclui_campo USING 'T_SAIDA' 'OBSERVACAO'       'OBSERVAÇÂO'             'C' '30' 'X' '' '' '',
    f_inclui_campo USING 'T_SAIDA' 'CALL_OBS'         'Anex.'                  'C' '5' 'X' ''  'X' '',
***Inserção - Hermiro Carvalho - RDM 196342 - 17/04/2018 - Fim
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
    f_inclui_campo USING 'T_SAIDA' 'WERKS'            'CENTRO'                 'C' '4' 'X' ''  '' '',
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Início
****Inserção - Hermiro Carvalho - RDM 181010 - 30/05/2017 - Início
    f_inclui_campo USING 'T_SAIDA' 'WRK02'            'CENTRO PRODUTIVO'       'C' '4' 'X' ''  '' '',
***Inserção - Hermiro Carvalho - RDM 181010 - 30/05/2017 - Início
    f_inclui_campo USING 'T_SAIDA' 'DESCRICAO'        'DESCRICAO MATERIAL'     'C' '40' ''  ''  '' '',
*RDM 203086
    f_inclui_campo USING 'T_SAIDA' 'LGORT'            'DEPÓSITO'               'C' '4' ''  ''  '' '',
*RDM 203086
    f_inclui_campo USING 'T_SAIDA' 'CLIENTE'          'CLIENTE'                'C' '10' ''  ''  '' '',
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Início
    f_inclui_campo USING 'T_SAIDA' 'COD_CLI'          'CÓDIGO CLIENTE'         'C' '0' ''  ''  '' '',

***Inicio - Fabio - 26.01.2021 - RDM 251520
    f_inclui_campo USING 'T_SAIDA' 'TP_CLIENTE'       'TIPO CLIENTE'           'C' '10' ''  ''  '' '',
***Inicio - Fabio - 26.01.2021 - RDM 251520

    f_inclui_campo USING 'T_SAIDA' 'CODIGO'           'MATERIAL CLIENTE'       'C' '35' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'PROJETO'          'PROJETO'                'C' '20' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'PESO_PECA'        'PESO PC(KG)'            'P' '18' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'AUSCH'            '% Refugo'               'P' '10' space space space space,
    f_inclui_campo USING 'T_SAIDA' 'FERRAMENTA'       'FERRAMENTA'             'C'  '5' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'ROTEIRO'          'ROTEIRO'                'C' '30' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'EMBALAGEM'        'EMBALAGEM'              'C' '30' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'QTD_RACK'         'QTD. RACKS'             'P' '18' ''  ''  '' '',"
    f_inclui_campo USING 'T_SAIDA' 'TAMANHO_LOTE'     'TAM. LOTE'              'C' '01' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_MINIMO'   'ESTOQUE MIN.'           'P' '18' ''  'X' '' '',
    f_inclui_campo USING 'T_SAIDA' 'EST_MIN_PEC'      'ESTOQUE MIN. PC'        'P' '18' ''  'X' '' '',"
*** Inicio da Alteração - Nelson Junior - Stefanini - 18.08.2025
    f_inclui_campo USING 'T_SAIDA' 'EST_MIN_DIAS'     'PROTOCOLO'              'P' '18' ''  'X' '' '',"
*** Inicio da Alteração - Nelson Junior - Stefanini - 18.08.2025
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Início
*    f_inclui_campo USING 'T_SAIDA' 'EST_MIN_DIAS'     'ESTOQUE MIN. DIAS'      'P' '18' ''  'X' '' '',"
***Inserção - Hermiro Carvalho - RDM 105472 - 29/07/2014 - Fim
    f_inclui_campo USING 'T_SAIDA' 'PREV_ORG'         'PREVISÃO ORG'           'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'PREV_ORG_PEC'     'PREVISÃO ORG PC'        'P' '18'  '' 'X' '' '',"

***Inicio - Fabio - 24.03.2021 - RDM 256631
    f_inclui_campo USING 'T_SAIDA' 'PREV_ORG_CEN'     'PREVISÃO ORG OUTRO CENTRO'    'P' '18'  '' 'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PREV_ORG_CEN_PEC' 'PREVISÃO ORG PC OUTRO CENTRO' 'P' '18'  '' 'X' '' '',"
***Fim - Fabio - 24.03.2021 - RDM 256631

    f_inclui_campo USING 'T_SAIDA' 'OV_EMITIDA'       'OVs EMITIDAS'           'P' '18'  '' 'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'OV_EMITIDA_PC'    'OVs EMITIDAS PC'        'P' '18'  '' 'X' '' '',

***Inserção - Hermiro Carvalho - RDM 137207 - 10/04/2015 - Início
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_1'        'PREVISÃO TN +1'            'P' '18'  '' '' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_1PC'      'PREVISÃO PC +1'            'P' '18'  '' '' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_2'        'PREVISÃO TN +2'            'P' '18'  '' '' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_2PC'      'PREVISÃO PC +2'            'P' '18'  '' '' '' '',"
***Inserção - Sidney Q de Andrade - RDM 164690 - 27/06/2016 - Inicio
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_3'        'PREVISÃO TN +3'            'P' '18'  '' '' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_3PC'      'PREVISÃO PC +3'            'P' '18'  '' '' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_4'        'PREVISÃO TN +4'            'P' '18'  '' '' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_4PC'      'PREVISÃO PC +4'            'P' '18'  '' '' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_5'        'PREVISÃO TN +5'            'P' '18'  '' '' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PERIODO_5PC'      'PREVISÃO PC +5'            'P' '18'  '' '' '' '',"
***Inserção - Sidney Q de Andrade - RDM 164690 - 27/06/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 137207 - 10/04/2015 - Fim
    f_inclui_campo USING 'T_SAIDA' 'APRODUZIR'        'A PRODUZIR'             'P' '10' ''  'X' '' '',
    f_inclui_campo USING 'T_SAIDA' 'APRODUZIR_PEC'    'A PRODUZIR PC'          'P' '18' ''  'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'PENDENCIA'        'PENDENCIA'              'P' '18' ''  'X' '' '',
    f_inclui_campo USING 'T_SAIDA' 'PENDENCIA_PEC'    'PENDENCIA PC'           'P' '18' ''  'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'OV'               'OV'                     'P' '18' ''  'X' '' '',
    f_inclui_campo USING 'T_SAIDA' 'OV_PECA'          'OV PC'                  'P' '18' ''  'X' '' '',"
***Inserção - Hermiro Carvalho - RDM 137207 - 10/04/2015 - Início
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Início
*    f_inclui_campo USING 'T_SAIDA' 'PLANEJADAS'       'PLANEJADAS'             'P' '18'  '' 'X' '',"
     f_inclui_campo USING 'T_SAIDA' 'PLANEJADAS'       'PLANEJADAS TON'         'P' '18'  '' 'X' '' '',"
     f_inclui_campo USING 'T_SAIDA' 'PLANEJADAS_PC'    'PLANEJADAS PC'          'P' '18'  '' 'X' '' '',"
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Início
    f_inclui_campo USING 'T_SAIDA' 'DATA_INICIO'       'DATA INICIO'            'D' '10'  '' '' '' '',"
***Inserção - Hermiro Carvalho - RDM 137207 - 10/04/2015 - Fim

***Inicio - Fabio - 14.08.2020 - RDM 238614
    f_inclui_campo USING 'T_SAIDA' 'VOLUME_REM'       'VOLUME REM'              'P' '18'  '' '' '' '',
    f_inclui_campo USING 'T_SAIDA' 'VOLUME_REM_PC'    'VOLUME REM PC'           'P' '18'  '' '' '' '',
    f_inclui_campo USING 'T_SAIDA' 'VOLUME_REM_1'     'VOLUME REM +1'           'P' '18'  '' '' '' '',
    f_inclui_campo USING 'T_SAIDA' 'VOLUME_REM_1_PC'  'VOLUME REM +1 PC'        'P' '18'  '' '' '' '',
    f_inclui_campo USING 'T_SAIDA' 'VOLUME_ICON'      'VOLS.'                   'C' '5' '' ''  'X' '',
***Fim - Fabio - 14.08.2020 - RDM 238614

*CENARIO BEXT - INICIO
    f_inclui_campo USING 'T_SAIDA' 'REQUISICOES'      'REQUISIÇÕES TON'         'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'REQUISICOES_PC'   'REQUISIÇÕES PC'          'P' '18' ''  'X' '' '',
***Inserção - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Início
*    f_inclui_campo USING 'T_SAIDA' 'PEDIDOS'          'PEDIDOS COMPRA'         'P' '18' ''  'X' '',
    f_inclui_campo USING 'T_SAIDA' 'PEDIDOS'          'PEDIDOS COMPRA TON'      'P' '18' ''  'X' 'X' '',
***Inserção - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Fim
***Inserção - Hermiro Carvalho - RDM 184185 - 19/09/2017 - Início
    f_inclui_campo USING 'T_SAIDA' 'FORNECEDOR'       'FORNECEDOR'              'C' '10' ''  '' '' '',
***Inserção - Hermiro Carvalho - RDM 184185 - 19/09/2017 - Fim
    f_inclui_campo USING 'T_SAIDA' 'PEDIDOS_PC'       'PEDIDOS COMPRA PC'       'P' '18' ''  'X' '' '',
*CENARIO BEXT - FIM

*PP - Criar coluna POs Outros Centros na GD - RDM 244981
    f_inclui_campo USING 'T_SAIDA' 'PEDIDOS_CEN'      'PEDIDOS COMPRA OUTRO CENTRO TON'      'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'PEDIDOS_CEN_PC'   'PEDIDOS COMPRA OUTRO CENTRO PC'       'P' '18' ''  'X' '' '',
*PP - Criar coluna POs Outros Centros na GD - RDM 244981

    f_inclui_campo USING 'T_SAIDA' 'OP_LIB'           'OP LIB.'                'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'OP_LIB_PEC'       'OP LIB. PC'             'P' '18' ''  'X' '' '', "

*RDM 190430 - 17.11.17 - INICIO
    f_inclui_campo USING 'T_SAIDA' 'OP_LIB_CEN'       'OP LIB. OUTRO CENTRO'    'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'OP_LIB_CEN_PEC'   'OP LIB. OUTRO CENTRO PC' 'P' '18' ''  'X' '' '',"
*RDM 190430 - 17.11.17 - FIM

    f_inclui_campo USING 'T_SAIDA' 'OP_ABER'          'OP ABER.'                'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'OP_ABER_PEC'      'OP ABER. PC'             'P' '18' ''  'X' '' '',"

*RDM 190430 - 17.11.17 - INICIO
    f_inclui_campo USING 'T_SAIDA' 'OP_ABER_CEN'      'OP ABER. OUTRO CENTRO'    'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'OP_ABER_CEN_PEC'  'OP ABER. OUTRO CENTRO PC' 'P' '18' ''  'X' '' '',"
*RDM 190430 - 17.11.17 - FIM

***Inserção - Hermiro Carvalho - RDM 137207 - 10/04/2015 - Início
    f_inclui_campo USING 'T_SAIDA' 'PRODUZIR_OP'      'PRODUZIR (OP)'          'P' '18' ''  'X' '' '',
    f_inclui_campo USING 'T_SAIDA' 'PRAZO'            'PRAZO'                  'D' '10' ''  '' '' '',
***Inserção - Hermiro Carvalho - RDM 137207 - 10/04/2015 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Início
*    f_inclui_campo USING 'T_SAIDA' 'EXPEDICAO'        'EXPEDICAO'              'P' '18' ''  'X' 'X',
*    f_inclui_campo USING 'T_SAIDA' 'EXPEDI_CEN'       'EXPEDICAO OUTRO CENTRO' 'P' '18' ''  'X' 'X',"
*    f_inclui_campo USING 'T_SAIDA' 'EXPEDICAO_PEC'    'EXPEDICAO PC'           'P' '18' ''  'X' '',"
    f_inclui_campo USING 'T_SAIDA' 'EXPEDICAO'        'EXPEDICAO'              'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'EXPEDICAO_PEC'    'EXPEDICAO PC'           'P' '18' ''  'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'EXPEDI_CEN'       'EXPEDICAO OUTRO CENTRO' 'P' '18' ''  'X' 'X' '',"
    f_inclui_campo USING 'T_SAIDA' 'EXPEDI_CEN_PC'    'EXPEDICAO OUTRO CENTRO PC' 'P' '18' ''  'X' '' '',"
*RDM 257471
    f_inclui_campo USING 'T_SAIDA' 'EXPEDI_CEN_DET'   'EXPEDICAO OUTRO CENTRO DET' 'C' '50' ''  '' '' '',"
*RDM 257471
***Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Fim
    f_inclui_campo USING 'T_SAIDA' 'EXP_INTER'        'EXP.MAT.INTERM.'        'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'EXP_INTER_PEC'    'EXP.MAT.INTERM. PC'     'P' '18' ''  'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'MAT_CQ'           'MAT. CQ'                'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'MAT_CQ_PEC'       'MAT. CQ PC'             'P' '18' ''  'X' '' '',"
***Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Início
*    f_inclui_campo USING 'T_SAIDA' 'FATURADO'         'FATURADO'               'P' '18' ''  'X' 'X',
*    f_inclui_campo USING 'T_SAIDA' 'FATURADO_CEN'     'FATURADO OUTRO CENTRO'  'P' '18' ''  'X' 'X',
*    f_inclui_campo USING 'T_SAIDA' 'FATURADO_PC'      'FATURADO PC'            'P' '18' ''  'X' '',"
    f_inclui_campo USING 'T_SAIDA' 'FATURADO'         'FATURADO(DT REM)'       'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'FATURADO1'        'FATURADO'               'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'FATURADO_PC'      'FATURADO(DT REM) PC'    'P' '18' ''  'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'FATURADO1_PC'     'FATURADO PC'            'P' '18' ''  'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'FATURADO_CEN'     'FATURADO(DT REM) OUTRO CENTRO'  'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'FATURADO_CEN_PC'  'FATURADO(DT REM) OUTRO CENTRO PC'  'P' '18' ''  'X' '' '',
    f_inclui_campo USING 'T_SAIDA' 'FATURADO1_CEN'    'FATURADO OUTRO CENTRO'  'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'FATURADO1_CEN_PC' 'FATURADO OUTRO CENTRO PC'  'P' '18' ''  'X' '' '',
***Inserção - Hermiro Carvalho - PRJ01313 - 14/10/2015 - Fim
    f_inclui_campo USING 'T_SAIDA' 'PRODTOT'          'PRODUCAO TOTAL'         'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'PRODTOT_PEC'      'PRODUCAO TOTAL PC'      'P' '18' ''  'X' '' '', "
***Inserção - Hermiro Carvalho - RDM 140238 - 22/06/2015 - Início
    f_inclui_campo USING 'T_SAIDA' 'NECESSIDADE_MP'   'NECESSIDADE MP'         'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'MATERIA_PRIMA'    'MATÉRIA PRIMA'          'P' '10' ''  ''  'X' '',"
***Inserção - Hermiro Carvalho - RDM 172372 - 30/11/2016 - Início
    f_inclui_campo USING 'T_SAIDA' 'DESC_MP'    'DESCRIÇÃO MP'          'C' '18' ''  ''  '' '',
***Inserção - Hermiro Carvalho - RDM 172372 - 30/11/2016 - Fim
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_MP'       'ESTOQUE MP (DEP. DO CLIENTE)'       'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_MP_OUTRAS'       'OUTRAS MPs'             'P' '18' ''  'X' 'X' '',
***Inserção - Hermiro Carvalho - RDM 185165 - 08/08/2017 - Início
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_MP_CEN'   'ESTOQUE MP OUTRO CENTRO' 'P' '18' ''  'X' 'X' '',
***Inserção - Hermiro Carvalho - RDM 185165 - 08/08/2017 - Fim

*RDM 257471
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_MP_CEN_DET'   'ESTOQUE MP OUTRO CENTRO DET' 'C' '50' ''  '' '' '',
*RDM 257471

*PP - Cenário Estoque Consignado - RDM 239896
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_CONSIGNADO' 'ESTOQUE MP (CONSIGNADO)'  'P' '18' ''  'X' 'X' '',
*PP - Cenário Estoque Consignado - RDM 239896

    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_LTAN'     'ESTOQUE MP (LISTA TÉC ANTIGA)'  'P' '18' ''  'X' 'X' '',
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_LTAL'     'ESTOQUE MP (ALTERNATIVO)'       'P' '18' ''  'X' 'X' '',
***Inserção - Hermiro Carvalho - RDM 140238 - 22/06/2015 - Fim
    f_inclui_campo USING 'T_SAIDA' 'SALDO_MP'         'SALDO MP'               'P' '18' ''  'X' '' '',
    f_inclui_campo USING 'T_SAIDA' 'ANALISE_PREV'     'ANÁLISE PREV.'          'P' '18' ''  'X' '' '',
    f_inclui_campo USING 'T_SAIDA' 'ANALISE_PREV_PEC' 'ANÁLISE PREV. PC'       'P' '18' ''  'X' '' '',"
    f_inclui_campo USING 'T_SAIDA' 'ANALISE'          'ANALISE'                'P' '18' ''  'X' '' '',
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
*    f_inclui_campo USING 'T_SAIDA' 'ABAST'            'PT. ABAST.(TON)'        'P' '18' ''  'X' '',"
*    f_inclui_campo USING 'T_SAIDA' 'ABAST_PEC'        'PT. ABAST.(PC)'         'P' '18' ''  'X' '',"
     f_inclui_campo USING 'T_SAIDA' 'ABAST'            'COBERTURA (DIAS)(TON)'        'P' '18' ''  'X' '' '',"
     f_inclui_campo USING 'T_SAIDA' 'ABAST_PEC'        'COBERTURA (DIAS)(PC)'        'P' '18' ''  'X' '' '',"
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim
*    f_inclui_campo USING 'T_SAIDA' 'ESTQ_MAX_TON'     'ESTOQUE MÁX TON'        'P' '10' ''  ''  '',"
*    f_inclui_campo USING 'T_SAIDA' 'ESTQ_MAX_PC'      'ESTOQUE MÁX PC'         'P' '10' ''  ''  ''."

    f_inclui_campo USING 'T_SAIDA' 'MAX_EXPE_TON'     'MÁX. EXPEDIÇÃO TON'     'P' '18' ''  ''  '' '',"
    f_inclui_campo USING 'T_SAIDA' 'MAX_EXPE_PC'      'MÁX. EXPEDIÇÃO PC'      'P' '18' ''  ''  '' ''. "
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
    PERFORM:
    f_inclui_campo USING 'T_SAIDA' 'DATA_MODIF'        'DATA ATUALIZAÇÂO'      'D' '10' ''  'X'  '' '',
    f_inclui_campo USING 'T_SAIDA' 'MEINS'             'UN. MEDIDA MAT.'       'C' '3' ''  'X'  '' '',
*    f_inclui_campo USING 'T_SAIDA' 'NIVEL_EST'         'NÌVEL ESTOQUE'         'C' '14' ''  'X'  '',
    f_inclui_campo USING 'T_SAIDA' 'H_MATERIAL'        'HIERARQ. MATER.'       'C' '10' ''  'X'  '' '',
    f_inclui_campo USING 'T_SAIDA' 'L_ENTREGA'         'LOCAL ENTREGA'         'C' '25' ''  'X'  '' '',
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Início
    f_inclui_campo USING 'T_SAIDA' 'ATIVO'             'ATIVO'                 'C' '1' ''  'X'  '' '',
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Início
    f_inclui_campo USING 'T_SAIDA' 'PRODUCAO'          'PRODUÇÃO'              'C' '18' ''  ''  '' ''.
***Inserção - Hermiro Carvalho - RDM 160097 - 13/04/2016 - Início
    PERFORM:
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Início
*    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_FORN'       'ESTOQUE NO FORNECEDOR'    'P' '10' ''  ''  '',
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_FORN'       'ESTOQUE NO FORNECEDOR'    'P' '10' ''  ''  'X' '',
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE_FORN_LTAN'  'ESTOQUE NO FORNECEDOR (LISTA TÉC ANTIGA)'    'P' '10' ''  ''  'X' '',
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Fim
*    f_inclui_campo USING 'T_SAIDA' 'EXPEDICAO_FORN'     'EXPEDICAO FORNECEDOR'     'P' '10' ''  ''  '',
*    f_inclui_campo USING 'T_SAIDA' 'MP_FORN'            'ESTOQUE MP FORNECEDOR'    'P' '10' ''  ''  '',
*    f_inclui_campo USING 'T_SAIDA' 'MP_FIS_FORN'        'ESTOQUE FISICO MP FORNECEDOR' 'P' '10' ''  ''  '',
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Início
*    f_inclui_campo USING 'T_SAIDA' 'ESTFAT'             'ESTOQUE + FATURADO'       'P' '10' ''  ''  '',
    f_inclui_campo USING 'T_SAIDA' 'ESTFAT'             'EXPEDICAO + FATURADO'      'P' '18' ''  ''  '' '',
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Fim
    f_inclui_campo USING 'T_SAIDA' 'VOL_APRODUZIR'      'VOLUME A PRODUZIR'        'P' '18' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'VOL_APROGRAMAR'     'VOLUME A PROGRAMAR'       'P' '18' ''  ''  '' '',
    f_inclui_campo USING 'T_SAIDA' 'ESTOQUE1DIA'        'ESTOQUE 1 DIA'            'P' '18' ''  ''  '' '',
*    f_inclui_campo USING 'T_SAIDA' 'PROCESSO_FORN'      'PROCESSO FORNECEDOR'      'P' '10' ''  ''  '',
***Inserção - Hermiro Carvalho - RDM 196641 - 06/08/2018 - Início
*    f_inclui_campo USING 'T_SAIDA' 'CEEX'               'COBERTURA ESTOQUE EXPEDIÇÃO' 'C' '19' ''  ''  '',
*    f_inclui_campo USING 'T_SAIDA' 'CFAT'               'COBERTURA FATURADO'       'C' '19' ''  ''  '',
*    f_inclui_campo USING 'T_SAIDA' 'CPDC'               'COBERTURA PEDIDOS COMPRA' 'C' '19' ''  ''  '',
*    f_inclui_campo USING 'T_SAIDA' 'CREQ'               'COBERTURA DE REQUISIÇÕES' 'C' '19' ''  ''  '',
*    f_inclui_campo USING 'T_SAIDA' 'CTOT'               'COBERTURA TOTAL'          'C' '19' ''  ''  '',
*    f_inclui_campo USING 'T_SAIDA' 'SCEE'               'STATUS COBERTURA ESTOQUE EXPEDIÇÃO' 'C' '19' ''  ''  ''.
     f_inclui_campo USING 'T_SAIDA' 'COBFAT_EXP_ORG'     'COBERTURA Fat+Expedição ORG.' 'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'COB_EXP_ORG'        'COBERTURA EXPEDIÇÃO ORG.'     'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'COB_FAT_ORG'        'COBERTURA FATURADO ORG.'      'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'COB_OP_TOTAL_DEM_ORG' 'COBERTURA OP TOTAL DEMANDA ORG.' 'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'COB_MP_FORNE_DEM_ORG' 'COBERTURA MP EM FORNECEDOR  DEMANDA ORG.' 'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'COB_PED_COMP_ORG'     'COBERTURA DE PEDIDOS COMPRA ORG.' 'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'COB_EXP_60'           'COBERTURA EXPEDIÇÃO (60 DIAS)' 'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'COBFAT_EXP_60'        'COBERTURA Fat+Expedição (60 DIAS)' 'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'COB_STAT_EXP_60'      'STATUS COBERTURA Expedição (60 DIAS)' 'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'STATUS_COBERT_EXP_ORG' 'STATUS COBERTURA Expedição ORG'      'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'VOL_MENSAL_ATENDIDO'   'VOLUME MENSAL ATENDIDO'              'C' '20' ''  ''  '' '',
     f_inclui_campo USING 'T_SAIDA' 'SOMA_MPS'              'SOMA DE MPS'                         'C' '20' ''  ''  '' '',
* >>> - 03.10.2019 14:06:46 - RDM 225838 - Simone
     f_inclui_campo USING 'T_SAIDA' 'ESTQTRAN'              'EM TRANSFERÊNCIA'                    'C' '9'  ''  ''  'X' ''.
* <<< - 03.10.2019 14:06:46 - RDM 225838 - Simone

    PERFORM f_inclui_campo USING 'ZRNPPT_PREV' 'STATUS_PCP' 'STATUS_PCP' 'C' '60' '' 'X' '' 'X'.
*    PERFORM f_inclui_campo USING 'ZRNPPT_PREV' 'STATUS_OM'  'STATUS_OM'  '' '60' '' 'X' '' 'X'.

    PERFORM f_inclui_campo USING 'T_SAIDA' 'QTD_MP_CHE' 'MP USINA CHEGAR'  'P' '18' '' 'X' 'X' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'DATA_ENTRE' 'DATA ENTR. USINA'  'D' '10' '' ''  ''  ''.
***Inserção - Hermiro Carvalho - RDM 196641 - 06/08/2018 - Fim
***Inserção - Hermiro Carvalho - RDM 160097 - 13/04/2016 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 138179 - 07/05/2015 - Fim
****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim

*RDM 274417 - RDM 281546
    PERFORM f_inclui_campo USING 'T_SAIDA'  'MATNR_MP_ALT_DET' 'MATERIAL - ESTOQUE MP(ALTERNATIVO)'  'C' '50' ''  ''  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'CHARG_MP_ALT_DET' 'LOTE - ESTOQUE MP(ALTERNATIVO)'      'C' '50' ''  ''  '' ''.
*RDM 274417 - RDM 281546

*** Início da Alteração - Silvia (UT00935) - 13.01.2023 - DEM:4972 / PRJ1353
    "Adicionar as informações das chamadas Kanban
    PERFORM f_inclui_campo USING 'T_SAIDA'  'CHMD_AG_PROD_PC' 'CHAMADAS AG. PROD.(PC)'    'P' '18' ''  'X'  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'CHMD_AG_PROD_TN' 'CHAMADAS AG. PROD.(TN)'    'P' '18' ''  'X'  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'CHMD_DT_NECES'   'DATA NECES. CHAMADAS'      'D' '15' ''  ''  '' ''.
*** Fim    da Alteração - Silvia (UT00935) - 13.01.2023 - DEM:4972 / PRJ1353
*RDM322490 - GVCA20241122 - Begin
    PERFORM f_inclui_campo USING 'T_SAIDA'  'STATUS_DEM'   'Status Demanda'       'C' '15' ''  ''  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'STATUS_ATEND' 'Status Atendimento'   'C' '20' ''  ''  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'RUPTURA_ESTQ' 'Ruptura de Estoque'   'D' '15' ''  ''  '' ''.
*RDM322490 - GVCA20241122 - End
*** Inicio da Alteração - Nelson Junior - Stefanini - 18.08.2025
    PERFORM f_inclui_campo USING 'T_SAIDA'  'TP_ATEND'          'Qualificação demanda'     'D' '15' ''  ''  '' ''.
*    PERFORM f_inclui_campo USING 'T_SAIDA'  'GIRO'              'Tipo de atendimento '     'D' '15' ''  ''  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'ESTOQUE_PA'        'Estoque Total PA (TON)'   'D' '15' ''  ''  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'LAMINACAO'         'Laminação'                'D' '15' ''  ''  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'PRODUTO'           'Produto'                  'D' '15' ''  ''  '' ''.

    "*** Início - Blend IT - Gilvan Silva(UZ28679) - RDM 339191 - 15.10.2026 11:48:32 - GPS
* RDM 335174  - Inicio - 27/02/2026
***    PERFORM f_inclui_campo USING 'T_SAIDA'  'dias_estoque'      'Dias Estoque'             'P' '06' ''  'X'  '' ''.
* RDM 335174  - Fim - 27/02/2026

    PERFORM f_inclui_campo USING 'T_SAIDA'  'DIAS_ESTOQUE'      'Dias Estoque'             'P' '06' ''  'X'  '' ''.
    "*** Fim - Blend IT - Gilvan Silva(UZ28679) - RDM 339191 - 15.10.2026 11:48:32 - GPS

    PERFORM f_inclui_campo USING 'T_SAIDA'  'NECESSIDADE_PROTC' 'Necessidade Protocolo'    'D' '15' ''  ''  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA'  'COBERTURA'         '% Cobertura'              'D' '15' ''  ''  '' ''.

  ENDIF.

***Inserção - LG - RDM 334177 - 15/04/2026 - Início - Campos EF
  IF p_corte IS NOT INITIAL.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZDEMADTOT'   'DEMANDA ATUAL TOTAL (ORG+TN1+TN2)'        'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZPRODTOT'    'EM PRODUÇÃO TOTAL'                          'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZEXPDTOT'    'EXPEDIÇÃO TOTAL'                            'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZFATTOT'     'FATURADO TOTAL'                             'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZPRODEXPFAT' 'PRODUÇÃO + EXPEDIÇÃO + FATURADO'            'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZDELTA'      'DELTA'                                      'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZOPSEMMOV'   'TOTAL DE OP SEM MOVIMENTAÇÃO'               'I' '10' '' 'X' 'X' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZPEDSEMMOV'  'TOTAL DE PEDIDO SEM MOVIMENTAÇÃO'           'I' '10' '' 'X' 'X' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZACAO'       'AÇÃO'                                       'C' '200' '' ''  '' ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZFATORG3'    'FATURADO ORG -3'                            'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZFATORG2'    'FATURADO ORG -2'                            'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZFATORG1'    'FATURADO ORG -1'                            'P' '18' '' 'X' ''  ''.
    PERFORM f_inclui_campo USING 'T_SAIDA' 'ZZFATTOTHIST' 'FATURADO TOTAL (HISTÓRICO)'                 'P' '18' '' 'X' ''  ''.
  ENDIF.
***Inserção - LG - RDM 334177 - 15/04/2026 - Fim
*** Fim da Alteração - Nelson Junior - Stefanini - 18.08.2025

ENDFORM.                    " F_MONTA_CAMPOS

*&---------------------------------------------------------------------*
*&      Form  F_MONTA_QUEBRA
*&---------------------------------------------------------------------*
FORM f_monta_quebra .

  IF op1 = 'X'.
    CLEAR gt_sort.
    gt_sort-fieldname = 'CLIENTE'.
    gt_sort-group     = 'X'.
    gt_sort-subtot    = ' '.
    gt_sort-up        = 'X'.
    APPEND gt_sort.
  ELSEIF op2 = 'X'.
    CLEAR gt_sort.
    gt_sort-fieldname = 'MATNR'.
    gt_sort-group     = 'X'.
    gt_sort-subtot    = ' '.
    gt_sort-up        = 'X'.
    APPEND gt_sort.
  ENDIF.

ENDFORM.                    " F_MONTA_QUEBRA

*&---------------------------------------------------------------------*
*&      Form  f_inclui_campo
*&---------------------------------------------------------------------*
FORM f_inclui_campo USING  VALUE(p_tabela) VALUE(p_campo) VALUE(p_titcol)
                           VALUE(p_tipo)   VALUE(p_tam)   VALUE(p_key)
                           VALUE(p_soma)   VALUE(p_hot)   VALUE(p_edit).

  CLEAR gt_fieldcat.
  gt_fieldcat-fieldname    = p_campo.           "Nome do campo
  gt_fieldcat-tabname      = p_tabela.          "Nome da tabela
  gt_fieldcat-reptext_ddic = p_titcol.          "Titulo da coluna
  gt_fieldcat-inttype      = p_tipo.            "Tipo de dado do campo
  gt_fieldcat-outputlen    = p_tam.             "Tamanho do campo
  gt_fieldcat-key          = p_key.             "Campo chave
  gt_fieldcat-do_sum       = p_soma.            "Somatório
* gt_fieldcat-no_zero      = 'X'.               "Colunas em branco sem zero
  gt_fieldcat-hotspot      = p_hot.
***Inserção - Hermiro Carvalho - RDM 196432 - 05/04/2018 - Início

  gt_fieldcat-edit         = p_edit.

  IF p_campo  EQ 'STATUS_OM'.
    gt_fieldcat-ref_tabname = 'ZRNPPT_PREV'.
  ENDIF.

  IF p_campo EQ 'STATUS_PCP'.
    gt_fieldcat-ref_tabname = 'ZRNPPT_PREV'.
  ENDIF.

  IF p_campo EQ 'CALL_OBS' OR
     p_campo EQ 'VOLUME_ICON'.                              "RDM 238614
    gt_fieldcat-icon         = 'X'.
  ENDIF.

***Inserção - Hermiro Carvalho - RDM 196432 - 05/04/2018 - Fim
  APPEND gt_fieldcat.

  ls_fieldcat-fieldname    = p_campo.           "Nome do campo
  ls_fieldcat-tabname      = 'T_SAIDA'.          "Nome da tabela
  ls_fieldcat-reptext      = p_titcol.          "Titulo da coluna
  ls_fieldcat-inttype      = p_tipo.            "Tipo de dado do campo
  ls_fieldcat-outputlen    = p_tam.             "Tamanho do campo
  ls_fieldcat-key          = p_key.             "Campo chave
  ls_fieldcat-do_sum       = p_soma.            "Somatório
* gt_fieldcat-no_zero      = 'X'.               "Colunas em branco sem zero
  ls_fieldcat-hotspot      = p_hot.

  APPEND ls_fieldcat TO lt_fcat.

ENDFORM.                    " f_inclui_campo

*&---------------------------------------------------------------------*
*&      Form  f_top_of_page
*&---------------------------------------------------------------------*
FORM f_top_of_page.

  DATA: lc_datahj(10).

* Imprime o título do relatório
  IF op1 = 'X'.
    CLEAR gs_header.
    gs_header-typ  = 'H'.
    gs_header-info = text-002.
    APPEND gs_header TO gt_header.
  ELSEIF op2 = 'X'.
    CLEAR gs_header.
    gs_header-typ  = 'H'.
    gs_header-info = text-003.
    APPEND gs_header TO gt_header.
  ENDIF.

* Imprime as informações do cabeçalho do relatório
*  CLEAR gs_header.
*  gs_header-typ  = 'S'.
*  CONCATENATE text-004 p_centro-low INTO gs_header-info SEPARATED BY space.
*  APPEND gs_header TO gt_header.

  IF s_cent_t-low IS NOT INITIAL.
    CLEAR t_cent_trab.
    READ TABLE t_cent_trab WITH KEY arbpl = s_cent_t-low.
    CLEAR gs_header.
    gs_header-typ  = 'S'.
    CONCATENATE text-005 s_cent_t-low t_cent_trab-ktext INTO gs_header-info SEPARATED BY space.
    APPEND gs_header TO gt_header.
  ENDIF.

  IF NOT p_period[] IS INITIAL.
    CLEAR gs_header.
    gs_header-typ  = 'S'.

    IF NOT p_period-high IS INITIAL AND p_period-high <> '99999999'.
      CONCATENATE text-006 p_period-low+6(2)'.' p_period-low+4(2) '.' p_period-low+0(4) ' até '
        p_period-high+6(2)'.' p_period-high+4(2) '.' p_period-high+0(4) INTO gs_header-info
        SEPARATED BY space.
    ELSE.
      CONCATENATE text-006 p_period-low+6(2)'.' p_period-low+4(2) '.' p_period-low+0(4)
      INTO gs_header-info SEPARATED BY space.
    ENDIF.
    APPEND gs_header TO gt_header.
  ENDIF.

  WRITE sy-datum TO lc_datahj.
  CLEAR gs_header.
  gs_header-typ  = 'S'.
  CONCATENATE text-007 lc_datahj INTO gs_header-info SEPARATED BY space.
  APPEND gs_header TO gt_header.

  CALL FUNCTION 'REUSE_ALV_COMMENTARY_WRITE'
    EXPORTING
      it_list_commentary = gt_header.

  FREE: gs_header,
        gt_header.
ENDFORM.                    "f_top_of_page

*---------------------------------------------------------------------*
*      Form  seleciona_cliente
*---------------------------------------------------------------------*
*      Selecionar Cliente
*---------------------------------------------------------------------*
FORM seleciona_cliente .

*PP - Correção erro Timeout - INC 3258470
  FREE r_bismt.
*PP - Correção erro Timeout - INC 3258470

  r_bismt-sign   = 'I'.
  r_bismt-option = 'EQ'.
  r_bismt-low    = p_clt.
  APPEND r_bismt.
  CLEAR r_bismt.

ENDFORM.                    " seleciona_cliente

*&---------------------------------------------------------------------*
*&      Form  f_seleciona_marc
*&---------------------------------------------------------------------*
FORM f_seleciona_marc .

* Busca estratégia do material
  SELECT matnr werks strgr minbe bstma shzet lgpro sobsl
    INTO TABLE t_marc
    FROM marc
    FOR ALL ENTRIES IN t_zrnppt_prev
    WHERE matnr = t_zrnppt_prev-matnr
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*      AND werks = p_centro-low.
      AND werks = t_zrnppt_prev-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim

  IF sy-subrc = 0.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
    SORT t_marc BY matnr werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
  ENDIF.

ENDFORM.                    " f_seleciona_marc

*&---------------------------------------------------------------------*
*&      Form  f_producao_total
*&---------------------------------------------------------------------*
FORM f_producao_total.

  DATA vl_tabix LIKE sy-tabix.
  DATA: v_werkscoa TYPE zrnppt_pcoa-werks,
        v_kdmat    TYPE vbap-kdmat.
  CLEAR: t_pmvm, t_pmvm[].

  IF NOT t_saida[] IS INITIAL.

    SELECT a~znumoa a~znumpp a~zmatnr a~zcharg a~zordvenda
           a~zitem a~zqtreg a~zqtreal a~zcodcliente a~zqtpeca b~werks a~zdatreg
      INTO TABLE t_pmvm
      FROM zrnppt_pmvm AS a INNER JOIN zrnppt_pcoa AS b
        ON a~znumoa = b~znumoa
      FOR ALL ENTRIES IN t_saida
* Bruno Quadros - CSC - 25.04.2011 - Inicio
*      WHERE "b~werks  = p_centro-low
      WHERE a~zmatnr = t_saida-matnr
* Bruno Quadros - CSC - 25.04.2011 - Fim
        AND a~zcodmov = '101'
        AND a~zdatreg IN p_period
        AND a~zflag  <> 'X'.

* Busca código de material do cliente para os lotes produzidos
    IF NOT t_pmvm[] IS INITIAL.

      LOOP AT t_pmvm.
        v_tabix = sy-tabix.
        CLEAR: vl_matcli, t_marc, v_werkscoa.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
        SELECT SINGLE werks
          INTO v_werkscoa
          FROM zrnppt_pcoa
         WHERE znumoa = t_pmvm-znumoa.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
        READ TABLE t_marc WITH KEY matnr = t_pmvm-zmatnr
* Bruno Quadros - CSC - 25.04.2011 - Inicio
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
*                                   werks = t_saida-wrk02
                                    werks = v_werkscoa
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
                                   BINARY SEARCH.
* Bruno Quadros - CSC - 25.04.2011 - Fim

        IF t_marc-strgr = '25'.

          CLEAR v_kdmat.
          SELECT SINGLE kdmat
            FROM vbap
            INTO v_kdmat
***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Início
*            WHERE vbeln = t_vbrp-aubel AND
*                  posnr = t_vbrp-aupos.
             WHERE vbeln = t_pmvm-zordvenda AND
                   posnr = t_pmvm-zitem.
***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Fim

          READ TABLE t_zrnppt_prev WITH KEY werks = v_werkscoa
                                        matnr     = t_pmvm-zmatnr
***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Início
                                        kunnr     = t_pmvm-zcodcliente
***Inserção - Hermiro Carvalho - PRJ01313 - 22/10/2015 - Fim
                                        codigo    = v_kdmat.
          IF sy-subrc EQ 0.
            t_pmvm-zmatcli = t_zrnppt_prev-codigo.
            t_pmvm-kdmat   = t_zrnppt_prev-codigo.
            MODIFY t_pmvm INDEX v_tabix.
          ENDIF.

        ENDIF.

        IF t_marc-strgr = '40'
* Inclusão - Rodrigo Soeiro - 12/2011 - Inicio
       OR  t_marc-strgr EQ '20' .
* Fim da Inclusão

***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
          READ TABLE t_zrnppt_prev WITH KEY werks = v_werkscoa
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
                                            matnr = t_pmvm-zmatnr.
          IF sy-subrc EQ 0.
            t_pmvm-zmatcli = t_zrnppt_prev-codigo.
            MODIFY t_pmvm INDEX v_tabix.
          ENDIF.

        ENDIF.
      ENDLOOP.

* Calculo do Produção TOTAL
      SORT t_pmvm BY zmatnr zmatcli zcodcliente.

      LOOP AT t_saida.
        vl_tabix = sy-tabix.
* Quando a estratégia é 25 não é necessário comparar o Código do Cliente para calcular a produção total
        IF t_saida-strgr <> '25'
* Inclusão - Rodrigo Soeiro - 12/2011 - Inicio
       AND t_marc-strgr <> '20'.
* Fim da Inclusão

          LOOP AT t_pmvm WHERE zmatnr = t_saida-matnr
                           AND werks  = t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
            CLEAR: v_werkscoa.
            SELECT SINGLE werks
              INTO v_werkscoa
              FROM zrnppt_pcoa
             WHERE znumoa = t_pmvm-znumoa
               AND werks  = t_saida-werks.
            IF sy-subrc IS NOT INITIAL.
              CONTINUE.
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
            IF t_pmvm-zqtreal > 0.
              ADD t_pmvm-zqtreal TO t_saida-prodtot.
            ELSE.
              ADD t_pmvm-zqtreg TO t_saida-prodtot.
            ENDIF.

            t_saida-prodtot_pec = t_saida-prodtot_pec + t_pmvm-zqtpeca * 1000.
          ENDLOOP.

        ELSE.

          LOOP AT t_pmvm WHERE zmatnr       = t_saida-matnr
                           AND zmatcli      = t_saida-codigo
                           AND zcodcliente  = t_saida-cod_cli
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Início
                           AND werks        = t_saida-werks.
            CLEAR: v_werkscoa.
            SELECT SINGLE werks
              INTO v_werkscoa
              FROM zrnppt_pcoa
             WHERE znumoa = t_pmvm-znumoa
               AND werks  = t_saida-werks.
            IF sy-subrc IS NOT INITIAL.
              CONTINUE.
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 126250 - 15/12/2014 - Fim
            IF t_pmvm-zqtreal > 0.
              ADD t_pmvm-zqtreal TO t_saida-prodtot.
            ELSE.
              ADD t_pmvm-zqtreg TO t_saida-prodtot.
            ENDIF.

            t_saida-prodtot_pec = t_saida-prodtot_pec + t_pmvm-zqtpeca * 1000.
          ENDLOOP.
        ENDIF.
        MODIFY t_saida INDEX vl_tabix.
      ENDLOOP.
    ENDIF.
  ENDIF.

ENDFORM.                    " f_producao_total

*&---------------------------------------------------------------------*
*&      Form  trata_peso_peca
*&---------------------------------------------------------------------*
FORM trata_peso_peca .
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
*  DATA: v_kdmat TYPE vbap-kdmat.
*
*  SELECT SINGLE kdmat
*    FROM vbap
*    INTO v_kdmat
*    WHERE vbeln = wa_plaf-kdauf AND
*          posnr = wa_plaf-kdpos.

  CLEAR: v_pni, t_tabela_ov[].

  IF wa_plaf-kdauf IS NOT INITIAL.

    CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
      EXPORTING
        input  = ' PDR_PARTNUMBER_INT'
      IMPORTING
        output = v_atinn.

    CALL FUNCTION 'ZRNSDF_SEARCH_COVD_MAE'
      EXPORTING
        ordem_venda              = wa_plaf-kdauf
        item_ordem               = wa_plaf-kdpos
        idioma                   = sy-langu
        caracteristica           = v_atinn
      TABLES
        itab_ov                  = t_tabela_ov
      EXCEPTIONS
        not_found                = 1
        caracteristica_not_found = 2
        ordem_venda_is_missing   = 3
        idioma_is_missing        = 4
        OTHERS                   = 5.
    IF sy-subrc EQ 0.

      READ TABLE t_tabela_ov INDEX 1.
      v_pni = t_tabela_ov-resul.
      IF v_pni = 'X'.
        CLEAR v_pni.
      ENDIF.
    ENDIF.

  ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
* calcular e testar a tolerância do peso da divisão de remessa

  IF wa_plaf-meins = 'KG'.

    wa_plaf-gsmng  =  wa_plaf-gsmng / 1000. " Qte da Ordem Plan.
    MOVE-CORRESPONDING wa_plaf TO t_plaf.
    MODIFY t_plaf INDEX v_tabix.

  ELSEIF wa_plaf-meins = 'ST' OR wa_plaf-meins = 'PC'.

    PERFORM trata_peca USING wa_plaf-matnr        "#7
                             v_pni.

    wa_plaf-gsmng =  v_qtde * wa_plaf-gsmng.
    MOVE-CORRESPONDING wa_plaf TO t_plaf.
    MODIFY t_plaf INDEX v_tabix.

  ENDIF.

ENDFORM.                    " trata_peso_peca

*&---------------------------------------------------------------------*
*&      Form  F_NECESSIDADE_MP
*&---------------------------------------------------------------------*
FORM f_necessidade_mp .

  DATA: lv_atinn      LIKE cabn-atinn,
        lv_partnumber TYPE zqmt_partnumber-partnumber.

  CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
    EXPORTING
      input  = 'SD_NUM_MAT1'
    IMPORTING
      output = lv_atinn.

  READ TABLE t_mara WITH KEY matnr = t_saida-matnr.

***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
  DATA: vl_peso           TYPE zrnppt_prev-peso,
        vl_org            TYPE zrnppt_prev-peso,
        vl_oplib          TYPE zrnppt_prev-peso,
        vl_tabix          TYPE sy-tabix,
        vl_exped          TYPE zrnppt_prev-peso,
        v_estoque_mp      TYPE mchb-clabs,
        v_estoque_mp_ltan TYPE mchb-clabs,
        v_estoque_mp_ltal TYPE mchb-clabs,
        v_estoque_calc    TYPE mchb-clabs,
        v_matnr           TYPE mchb-matnr,
        v_werks           TYPE mchb-werks,
        v_lgort           TYPE mchb-lgort,
        v_charg           TYPE mchb-charg,
        v_clabs           TYPE mchb-clabs,
        v_idnrk           TYPE zqmt_partn_mrp-idnrk,
        v_aux_kdmat       TYPE char35 VALUE IS INITIAL.

  DATA: lv_saldoest     TYPE  labst,
        lv_qtde_reserva TYPE  labst,
        lv_qtde_estoque TYPE  labst,
        lv_soma         TYPE  labst.

***Inserção - Hermiro Carvalho - RDM 105472 - 14/08/2014 - Fim
  CLEAR: vl_peso, vl_org, vl_oplib, v_neces_mp, v_wmeng.

*  RANGES: r_lgort FOR tvarvc-low.
  DATA: BEGIN OF t_low OCCURS 0,
          low LIKE tvarvc-low,
        END OF t_low.

* Verifica depositos que não são analisados
  CLEAR: r_lgort.
*PP - Correção erro Timeout - INC 3258470
  FREE r_lgort.
*PP - Correção erro Timeout - INC 3258470
  SELECT low
    INTO TABLE t_low
    FROM tvarvc
   WHERE name = 'ZDEP_DERIVADO'.

  r_lgort-sign   = 'I'.
  r_lgort-option = 'EQ'.

  LOOP AT t_low.
    r_lgort-low = t_low-low.
    APPEND r_lgort.
  ENDLOOP.

  v_aux_kdmat = t_saida-codigo.

* Verifica unidade de medida
  PERFORM trata_peca USING t_saida-matnr            "#8
                           t_saida-pn_interno.

* Busca lista técnica
*  SELECT SINGLE *
*    FROM mast
*    WHERE matnr = t_saida-matnr
*      AND werks = t_saida-wrk02.
  READ TABLE t_mast_log INTO wa_mast_log WITH KEY matnr = t_saida-matnr
                                                  werks = t_saida-wrk02.

  IF sy-subrc EQ 0.
*    SELECT stlnr idnrk meins menge
*      INTO TABLE t_stpo_aux
*      FROM stpo
*     WHERE stlnr EQ mast-stlnr.
    READ TABLE t_stpo_log INTO wa_stpo_log WITH KEY stlnr = wa_mast_log-stlnr.
    IF sy-subrc EQ 0.

      t_saida-materia_mp = wa_stpo_log-idnrk.

      IF t_mara-meins = 'PC' OR t_mara-meins = 'ST'.
        v_wmeng = t_saida-aproduzir_pec.
      ELSE.
        v_wmeng = t_saida-aproduzir.
      ENDIF.
      CLEAR: v_neces_mp, v_neces_mp_aux.

*      LOOP AT t_stpo_aux INTO wa_stpo_aux.
*
*        IF  NOT t_mara-kzkfg IS INITIAL.
*
*          CLEAR: v_idnrk.
*          SELECT SINGLE idnrk
*            INTO v_idnrk
*            FROM zqmt_partn_mrp
*           WHERE partnumber = t_saida-pn_interno
*             AND werks = t_saida-werks.
*
*          wa_stpo_aux-idnrk = v_idnrk.
*
*          MODIFY t_stpo_aux FROM wa_stpo_aux.
*
*        ENDIF.
*
*        IF wa_stpo_aux-meins = 'KG'.
*          v_neces_mp_aux = ( wa_stpo_aux-menge / 1000 ) * v_wmeng.
*          v_neces_mp     = v_neces_mp + v_neces_mp_aux.
*        ELSE.
*          v_neces_mp_aux = wa_stpo_aux-menge * v_wmeng.
*          v_neces_mp     = v_neces_mp + v_neces_mp_aux.
*        ENDIF.
*
*      ENDLOOP.

      LOOP AT t_stpo_log INTO wa_stpo_log WHERE stlnr = wa_mast_log-stlnr.
        IF wa_stpo_log-meins = 'KG'.
          v_neces_mp_aux = ( wa_stpo_log-menge / 1000 ) * v_wmeng.
          v_neces_mp     = v_neces_mp + v_neces_mp_aux.
        ELSE.
          v_neces_mp_aux = wa_stpo_log-menge * v_wmeng.
          v_neces_mp     = v_neces_mp + v_neces_mp_aux.
        ENDIF.
      ENDLOOP.

* Se total a produzir for negativo, o campo necessidade MP fica igual á zero.
      IF t_saida-aproduzir LE 0.
        t_saida-necessidade_mp = 0.
      ELSE.
        t_saida-necessidade_mp = v_neces_mp.
      ENDIF.

*      t_saida-materia_mp = wa_stpo_aux-idnrk.

* Saldo da Matéria_prima
      CLEAR: v_saldo, v_cq, v_saldo_mp.
*      CLEAR: r_lgort.
*
*      SELECT low
*        INTO TABLE t_low
*        FROM tvarvc
*       WHERE name = 'ZDEP_DERIVADO'.
*
*      r_lgort-sign   = 'I'.
*      r_lgort-option = 'EQ'.
*
*      LOOP AT t_low.
*        r_lgort-low = t_low-low.
*        APPEND r_lgort.
*      ENDLOOP.

      CLEAR vl_tabix.

*      LOOP AT t_stpo_aux INTO wa_stpo_aux.
*
*        CLEAR vl_tot_neces_mp.
*
*        LOOP AT t_saida INTO wa_saida WHERE materia_mp EQ wa_stpo_aux-idnrk
*                                        AND werks      EQ t_saida-werks.
*          vl_tot_neces_mp = vl_tot_neces_mp + wa_saida-necessidade_mp.
*        ENDLOOP.
*      ENDLOOP.

      CLEAR vl_tot_neces_mp.
      LOOP AT t_stpo_log INTO wa_stpo_log WHERE stlnr = wa_mast_log-stlnr.

        LOOP AT t_saida INTO wa_saida WHERE materia_mp EQ wa_stpo_log-idnrk
                                        AND werks      EQ t_saida-werks.
          vl_tot_neces_mp = vl_tot_neces_mp + wa_saida-necessidade_mp.
        ENDLOOP.

      ENDLOOP.

*Apurar Estoque MP
      CLEAR: v_estoque_mp, t_alternativa[], wa_alternativa, t_mchb_aux[],
             wa_mchb_aux, v_estoque_mp_ltan, v_estoque_mp_ltal,
             t_hist_lt[], wa_hist_lt.

*     Início alteração - RDM 266927 - 24.09.2021
*      LOOP AT t_stpo_log INTO wa_stpo_log WHERE stlnr = wa_mast_log-stlnr.
*        CLEAR v_estoque_mp.
*        SELECT SUM( clabs )
*          INTO (v_estoque_mp) FROM mchb
*          WHERE matnr     EQ wa_stpo_log-idnrk
*          AND werks       EQ t_saida-werks
*          AND lgort  NOT IN r_lgort
*          AND mchb~clabs  > 0.
*        t_saida-estoque_mp = v_estoque_mp + t_saida-estoque_mp.
*      ENDLOOP.

      LOOP AT t_stpo_log INTO wa_stpo_log WHERE stlnr = wa_mast_log-stlnr.
        CLEAR v_estoque_mp.
        SELECT matnr werks lgort charg clabs
          INTO (v_matnr, v_werks, v_lgort, v_charg, v_estoque_mp) FROM mchb
          WHERE matnr  EQ wa_stpo_log-idnrk
            AND werks  EQ t_saida-werks
            AND lgort  NOT IN r_lgort
            AND clabs  GT 0.
*INC 3708419
          PERFORM f_check_disponibilidade_lote USING  v_matnr
                                                      v_werks
                                                      v_lgort
                                                      v_charg
                                            CHANGING  v_estoque_mp.
          IF v_estoque_mp LE 0.
            CONTINUE.
          ENDIF.
*INC 3708419

          IF v_lgort EQ t_saida-lgort.
            t_saida-estoque_mp = v_estoque_mp + t_saida-estoque_mp.
          ELSE.
            t_saida-estoque_mp_outras = v_estoque_mp + t_saida-estoque_mp_outras.
          ENDIF.
        ENDSELECT.
      ENDLOOP.
*     Fim alteração - RDM 266927 - 24.09.2021

***Inserção - Hermiro Carvalho - RDM 185165 - 08/08/2017 - Início
      LOOP AT t_stpo_log INTO wa_stpo_log WHERE stlnr = wa_mast_log-stlnr.
        CLEAR: v_estoque_mp.
        SELECT matnr werks lgort charg clabs
          INTO (v_matnr, v_werks, v_lgort, v_charg, v_estoque_mp)
          FROM mchb
          WHERE matnr  EQ wa_stpo_log-idnrk
            AND werks  NE t_saida-werks
            AND lgort  NOT IN r_lgort
            AND clabs  GT 0.
*INC 3708419
          PERFORM f_check_disponibilidade_lote USING  v_matnr
                                                      v_werks
                                                      v_lgort
                                                      v_charg
                                            CHANGING  v_estoque_mp.
          IF v_estoque_mp LE 0.
            CONTINUE.
          ENDIF.
*INC 3708419
          t_saida-estoque_mp_cen = v_estoque_mp + t_saida-estoque_mp_cen.
        ENDSELECT.
      ENDLOOP.
***Inserção - Hermiro Carvalho - RDM 185165 - 08/08/2017 - Fim
*Apurar Estoque MP (Lista Técnica Antiga)
      SELECT werks matnr idnrk
        INTO TABLE t_hist_lt
        FROM zsuppt_hist_lt
       WHERE werks      = t_saida-wrk02
         AND matnr      = t_saida-matnr.

      IF t_hist_lt[] IS NOT INITIAL.
        SELECT matnr werks lgort charg clabs ersda cinsm
          INTO CORRESPONDING FIELDS OF TABLE t_mchb_aux
          FROM mchb
           FOR ALL ENTRIES IN t_hist_lt
         WHERE matnr EQ t_hist_lt-idnrk
           AND werks EQ t_hist_lt-werks
           AND lgort NE r_lgort
           AND clabs > 0.
        IF t_mchb_aux[] IS NOT INITIAL.
          LOOP AT t_mchb_aux INTO wa_mchb_aux.
*INC 3708419
            PERFORM f_check_disponibilidade_lote USING  wa_mchb_aux-matnr
                                                        wa_mchb_aux-werks
                                                        wa_mchb_aux-lgort
                                                        wa_mchb_aux-charg
                                              CHANGING  wa_mchb_aux-clabs.
*INC 3708419
            v_estoque_mp_ltan = v_estoque_mp_ltan + wa_mchb_aux-clabs.
            CLEAR: wa_mchb_aux.
          ENDLOOP.
        ENDIF.
      ENDIF.
      t_saida-estoque_ltan = v_estoque_mp_ltan.

*Apurar Estoque MP (Alternativo)
      CLEAR: t_mchb_aux[], wa_mchb_aux.
      SELECT werks matnr charg matnr_pa clabs rsnum
        INTO CORRESPONDING FIELDS OF TABLE t_alternativa
        FROM zsuppt_lista_alt


        "*** Início - Blend IT - Gilvan Silva(UZ28679) - RDM 339191 - 15.10.2026 16:14:39 - GPS
        WHERE werks     = t_saida-wrk02
          AND matnr_pa  = t_saida-matnr
        "*** Fim - Blend IT - Gilvan Silva(UZ28679) - RDM 339191 - 15.10.2026 16:14:39 - GPS


*PP - Otimização de Estoque - RDM 255762
          AND desativada  = ''.
*PP - Otimização de Estoque - RDM 255762
      IF sy-subrc IS INITIAL.
        SELECT werks lgort matnr charg clabs ersda cinsm
          INTO CORRESPONDING FIELDS OF TABLE t_mchb_aux
          FROM mchb
          FOR ALL ENTRIES IN t_alternativa
          WHERE matnr EQ t_alternativa-matnr
            AND charg EQ t_alternativa-charg
            AND lgort NE r_lgort
            AND clabs > 0.
        SORT t_mchb_aux BY matnr charg.
*INC 3708419
        LOOP AT t_mchb_aux INTO wa_mchb_aux.
          vl_tabix = sy-tabix.
          PERFORM f_check_disponibilidade_lote USING  wa_mchb_aux-matnr
                                                      wa_mchb_aux-werks
                                                      wa_mchb_aux-lgort
                                                      wa_mchb_aux-charg
                                            CHANGING  wa_mchb_aux-clabs.
          IF wa_mchb_aux-clabs LE 0.
            DELETE t_mchb_aux INDEX vl_tabix.
          ENDIF.
        ENDLOOP.
*INC 3708419
        IF t_mchb_aux[] IS NOT INITIAL.

          LOOP AT t_alternativa INTO wa_alternativa.

            vl_tabix = sy-tabix.
            READ TABLE t_mchb_aux INTO wa_mchb_aux WITH KEY matnr = wa_alternativa-matnr
                                                            charg = wa_alternativa-charg.

            IF sy-subrc NE 0 OR ( sy-subrc = 0 AND wa_mchb_aux-clabs = 0 ).
              DELETE t_alternativa INDEX vl_tabix.
            ENDIF.

          ENDLOOP.

*PP - Melhorias MP Alternativa - RDM 271191
          DATA: lt_pajt_log TYPE TABLE OF zppt_pajt_log,
                ls_pajt_log TYPE zppt_pajt_log.

          SELECT werks matnr charg werks_apl matnr_apl
                 udate utime clabs
          INTO CORRESPONDING FIELDS OF TABLE lt_pajt_log
          FROM zppt_pajt_log
            FOR ALL ENTRIES IN t_mchb_aux
          WHERE werks EQ t_mchb_aux-werks
            AND matnr EQ t_mchb_aux-matnr
            AND charg EQ t_mchb_aux-charg
            AND werks_apl EQ t_saida-wrk02
            AND matnr_apl EQ t_saida-matnr
            AND situacao EQ 'APROVADO'.

          IF sy-subrc IS NOT INITIAL. "Se o registro não for encontrado na tab. zppt_pajt_log, Leia tabela ZPPT_OEST_LOG e inclui na tabela it_pajt_log. Flávio Marqueviski

            SELECT werks matnr charg werks_apl matnr_apl
              udate utime clabs
            INTO CORRESPONDING FIELDS OF TABLE lt_pajt_log
            FROM zppt_oest_log
              FOR ALL ENTRIES IN t_mchb_aux
            WHERE werks EQ t_mchb_aux-werks
              AND matnr EQ t_mchb_aux-matnr
              AND charg EQ t_mchb_aux-charg
              AND werks_apl EQ t_saida-wrk02
              AND matnr_apl EQ t_saida-matnr
              AND situacao EQ 'APROVADO'.

            CLEAR: t_mseg, t_mseg_sum.
            SELECT m~werks m~matnr m~charg m~bwart m~menge m~aufnr a~matnr
              FROM mseg AS m
              INNER JOIN afpo AS a
              ON a~aufnr = m~aufnr
              INTO TABLE t_mseg
              FOR ALL ENTRIES IN t_alternativa
              WHERE m~matnr = t_alternativa-matnr
                AND m~charg = t_alternativa-charg
                AND a~matnr = t_alternativa-matnr_pa
                AND ( m~bwart = '261' OR m~bwart = '262' ).

            LOOP AT t_mseg INTO wa_mseg.

              IF wa_mseg-bwart = '262'.
                wa_mseg-menge = wa_mseg-menge * -1.
              ENDIF.

              CLEAR: wa_mseg-bwart, wa_mseg-aufnr,  wa_mseg-matnr_apl.

              COLLECT wa_mseg INTO t_mseg_sum.

            ENDLOOP.

          ENDIF.

          SORT: lt_pajt_log BY matnr charg DESCENDING,
                t_mseg_sum  BY matnr charg.
*PP - Melhorias MP Alternativa - RDM 271191

          "*** Início - Blend IT - Gilvan Silva(UZ28679) - RDM 339191 - 15.10.2026 16:19:15 - GPS
        ELSE.
          REFRESH t_alternativa.
          "*** Fim - Blend IT - Gilvan Silva(UZ28679) - RDM 339191 - 15.10.2026 16:19:15 - GPS

        ENDIF.

        CLEAR v_estoque_mp_ltal.
        SORT t_alternativa BY werks matnr charg.
        "***DELETE ADJACENT DUPLICATES FROM t_alternativa COMPARING werks matnr charg.                "*** Blend IT - Gilvan Silva(UZ28679) - RDM 339191 - 23.10.2026 17:58:50 - GPS
        DELETE ADJACENT DUPLICATES FROM t_alternativa COMPARING werks matnr charg clabs.              "*** Blend IT - Gilvan Silva(UZ28679) - RDM 339191 - 23.10.2026 17:58:50 - GPS


*        LOOP AT t_alternativa INTO wa_alternativa.
*
*          CLEAR v_estoque_calc.
*
*          READ TABLE t_mseg_sum INTO wa_mseg_sum WITH KEY matnr = wa_alternativa-matnr
*                                                          charg = wa_alternativa-charg BINARY SEARCH.
*          IF sy-subrc EQ 0.
*            v_estoque_calc = wa_alternativa-clabs - wa_mseg_sum-menge.
*          ELSE.
*            v_estoque_calc = wa_alternativa-clabs.
*          ENDIF.
*
*          READ TABLE t_mchb_aux INTO wa_mchb_aux WITH KEY matnr = wa_alternativa-matnr
*                                                          charg = wa_alternativa-charg BINARY SEARCH.
*          IF sy-subrc EQ 0.
*
*            IF wa_mchb_aux-clabs < v_estoque_calc.
*              v_estoque_mp_ltal = v_estoque_mp_ltal + wa_mchb_aux-clabs.
*            ELSE.
*              v_estoque_mp_ltal = v_estoque_mp_ltal + v_estoque_calc.
*            ENDIF.
*
*          ELSE.
*            v_estoque_mp_ltal = v_estoque_mp_ltal + v_estoque_calc.
*          ENDIF.
*
*        ENDLOOP.
        DATA(t_alternativa_aux) = t_alternativa[].
        SORT t_alternativa_aux BY matnr.
        DELETE ADJACENT DUPLICATES FROM t_alternativa_aux COMPARING matnr.

        IF t_alternativa_aux[] IS NOT INITIAL.
          SELECT matnr, meins
            INTO TABLE @DATA(lt_mara)
              FROM mara
                FOR ALL ENTRIES IN @t_alternativa_aux
                WHERE matnr = @t_alternativa_aux-matnr.
        ENDIF.

        CLEAR v_estoque_mp_ltal.
        LOOP AT t_alternativa INTO wa_alternativa.

          CALL FUNCTION 'ZPPF_SALDO_ALTERNATIVO'
            EXPORTING
              iv_matnr        = wa_alternativa-matnr
              iv_charg        = wa_alternativa-charg
              iv_matnt_apl    = wa_alternativa-matnr_pa
              iv_rsnum        = wa_alternativa-rsnum
            IMPORTING
              ev_saldoest     = lv_saldoest
              ev_qtde_reserva = lv_qtde_reserva
              ev_qtde_estoque = lv_qtde_estoque.

          lv_soma = lv_qtde_reserva + lv_qtde_estoque.
          v_estoque_mp_ltal = wa_alternativa-clabs - lv_soma.
          IF v_estoque_mp_ltal < 0.
            v_estoque_mp_ltal = 0.
          ENDIF.

          IF lv_saldoest < v_estoque_mp_ltal.
            v_estoque_mp_ltal = lv_saldoest.
          ENDIF.

          TRY.
              DATA(ls_mara) = lt_mara[ matnr = wa_alternativa-matnr ].
              IF NOT ls_mara-meins = 'TO'.
                CALL FUNCTION 'MD_CONVERT_MATERIAL_UNIT'
                  EXPORTING
                    i_matnr              = ls_mara-matnr
                    i_in_me              = ls_mara-meins
                    i_out_me             = 'TO'
                    i_menge              = v_estoque_mp_ltal
                  IMPORTING
                    e_menge              = v_estoque_mp_ltal
                  EXCEPTIONS
                    error_in_application = 1
                    error                = 2
                    OTHERS               = 3.
              ENDIF.
            CATCH cx_sy_itab_line_not_found.
          ENDTRY.

          t_saida-estoque_ltal = t_saida-estoque_ltal + v_estoque_mp_ltal.

        ENDLOOP.

      ENDIF.
*      t_saida-estoque_ltal = v_estoque_mp_ltal.

*RDM 274417
      DATA lv_tabix LIKE sy-tabix.
      LOOP AT t_alternativa INTO wa_alternativa.
        lv_tabix = sy-tabix.
        READ TABLE t_mchb_aux INTO wa_mchb_aux WITH KEY matnr = wa_alternativa-matnr
                                                        charg = wa_alternativa-charg.
        IF sy-subrc IS NOT INITIAL.
          DELETE t_alternativa INDEX lv_tabix.
        ENDIF.
      ENDLOOP.

      IF lines( t_alternativa ) GT 4.
        t_saida-charg_mp_alt_det = 'VARIOS'.
      ELSE.
        LOOP AT t_alternativa INTO wa_alternativa.
          IF t_saida-charg_mp_alt_det IS INITIAL.
            t_saida-charg_mp_alt_det = wa_alternativa-charg.
          ELSE.
            CONCATENATE t_saida-charg_mp_alt_det wa_alternativa-charg INTO t_saida-charg_mp_alt_det SEPARATED BY '/'.
          ENDIF.
        ENDLOOP.
      ENDIF.

      SORT t_alternativa BY matnr.
      DELETE ADJACENT DUPLICATES FROM t_alternativa COMPARING matnr.
      IF lines( t_alternativa ) GT 4.
        t_saida-matnr_mp_alt_det = 'VARIOS'.
      ELSE.
        LOOP AT t_alternativa INTO wa_alternativa.
          IF t_saida-matnr_mp_alt_det IS INITIAL.
            t_saida-matnr_mp_alt_det = wa_alternativa-matnr.
          ELSE.
            CONCATENATE t_saida-matnr_mp_alt_det wa_alternativa-matnr INTO t_saida-matnr_mp_alt_det SEPARATED BY '/'.
          ENDIF.
        ENDLOOP.
      ENDIF.
*RDM 274417

*Saldo MP
      vl_tot_neces_mp   = vl_tot_neces_mp + v_neces_mp.
      t_saida-saldo_mp  = ( t_saida-estoque_mp + t_saida-estoque_ltan + t_saida-estoque_ltal ) - 	vl_tot_neces_mp.

    ENDIF.
***INÍCIO - Sandra H. DM 178038 - 27/03/2017 - Início
*** Apurar estoque MP para venda de in-natura
  ELSE.

    IF wa_stpo_aux-idnrk IS INITIAL.
      SELECT werks lgort matnr charg clabs ersda cinsm
        INTO CORRESPONDING FIELDS OF TABLE t_mchb_aux2
        FROM mchb
          WHERE matnr     EQ t_saida-matnr
          AND werks       EQ t_saida-werks
          AND lgort  NOT IN r_lgort
          AND mchb~clabs  > 0.

      LOOP AT t_mchb_aux2 INTO wa_mchb_aux.
*INC 3708419
        PERFORM f_check_disponibilidade_lote USING  wa_mchb_aux-matnr
                                                    wa_mchb_aux-werks
                                                    wa_mchb_aux-lgort
                                                    wa_mchb_aux-charg
                                          CHANGING  wa_mchb_aux-clabs.
        IF wa_mchb_aux-clabs LE 0.
          CONTINUE.
        ENDIF.
*INC 3708419

        t_saida-estoque_mp = wa_mchb_aux-clabs + t_saida-estoque_mp.
      ENDLOOP.

      PERFORM apura_saldo_mp.

      vl_tot_neces_mp   = vl_tot_neces_mp + v_neces_mp.
      t_saida-saldo_mp  = t_saida-estoque_mp  -   vl_tot_neces_mp.

    ENDIF.
***FIM - Sandra H. DM 178038 - 27/03/2017 - Início

  ENDIF.
*  ENDIF.

  vl_peso  = t_saida-op_lib + t_saida-op_aber.
  vl_org   = ( t_saida-prev_org / 30 ) * 10.
  vl_oplib = t_saida-op_lib + t_saida-op_aber.

  CLEAR: vl_exped.
  vl_exped = t_saida-expedicao + t_saida-expedi_cen.

  IF t_saida-saldo_mp <= '0.000'.
    t_saida-nivel_est = 'SEM MP'.
  ENDIF.

  IF t_saida-saldo_mp <= '0.000' AND ( vl_exped = t_saida-estoque_minimo OR
                                     t_saida-estoque_minimo <= '0.000').
    t_saida-nivel_est = 'SEM MP C/ MIN'.
  ENDIF.

  IF t_saida-saldo_mp > '0.000' AND
                        ( vl_peso  <= '0.000')  AND ( vl_exped <= '0.000' ).
    t_saida-nivel_est = 'SEM OP'.
  ENDIF.

  IF t_saida-saldo_mp > '0.000' AND ( vl_oplib > '0.000' ) AND
               ( vl_exped <= vl_org ).
    t_saida-nivel_est = 'A PRODUZIR'.
  ENDIF.

  IF vl_exped >= t_saida-estoque_minimo.
    t_saida-nivel_est = 'MÍNIMO'.
  ENDIF.

*RDM 257471
  DATA: BEGIN OF lit_estoque_mp_cen OCCURS 0,
          matnr TYPE mchb-matnr,    "*INC 3708419
          werks TYPE mchb-werks,
          lgort TYPE mchb-lgort,    "*INC 3708419
          charg TYPE mchb-charg,    "*INC 3708419
          clabs TYPE mchb-clabs,    "*INC 3708419
        END OF lit_estoque_mp_cen.

  CLEAR t_saida-estoque_mp_cen_det.

  SELECT matnr werks lgort charg clabs
    INTO TABLE lit_estoque_mp_cen
    FROM mchb
    WHERE matnr  EQ t_saida-materia_mp
      AND werks  NE t_saida-werks
      AND lgort  NOT IN r_lgort
      AND mchb~clabs  > 0.
*INC 3708419
  LOOP AT lit_estoque_mp_cen.
    vl_tabix = sy-tabix.
    PERFORM f_check_disponibilidade_lote USING  lit_estoque_mp_cen-matnr
                                                lit_estoque_mp_cen-werks
                                                lit_estoque_mp_cen-lgort
                                                lit_estoque_mp_cen-charg
                                      CHANGING  lit_estoque_mp_cen-clabs.
    IF lit_estoque_mp_cen-clabs LE 0.
      DELETE lit_estoque_mp_cen INDEX vl_tabix.
    ENDIF.
  ENDLOOP.
*INC 3708419

  SORT lit_estoque_mp_cen BY werks ASCENDING.
  DELETE ADJACENT DUPLICATES FROM lit_estoque_mp_cen COMPARING werks.
  LOOP AT lit_estoque_mp_cen.
    IF t_saida-estoque_mp_cen_det IS INITIAL.
      t_saida-estoque_mp_cen_det = lit_estoque_mp_cen-werks.
    ELSE.
      CONCATENATE t_saida-estoque_mp_cen_det lit_estoque_mp_cen-werks INTO t_saida-estoque_mp_cen_det SEPARATED BY '-'.
    ENDIF.
  ENDLOOP.

  FREE lit_estoque_mp_cen.
*RDM 257471

ENDFORM.                    " F_NECESSIDADE_MP

*INC 3097794
*&---------------------------------------------------------------------*
*&      Form  F_APPEND_MAT_INTERMEDIARIO
*&---------------------------------------------------------------------*
FORM f_append_mat_intermediario.

  DATA: t_makt           TYPE TABLE OF makt WITH HEADER LINE,
        v_prev_org       TYPE zrnppt_prev-peso,
        v_estoque_minimo LIKE zrnppt_prev-estoque_minimo.
  DATA: v_estoque_min_pc LIKE zrnppt_prev-estoque_min_pc.
  DATA: v_atflv    TYPE atflv,
        v_cuobj    LIKE inob-cuobj,
        v_converte TYPE p DECIMALS 2,
        v_idnrk    TYPE zqmt_partn_mrp-idnrk,
        v_werks    TYPE aufk-werks.

  DATA t_stpo_aux LIKE TABLE OF t_stpo WITH HEADER LINE.

  CLEAR: wa_saida, mast-stlnr, t_stpo, t_makt[], v_prev_org, v_estoque_minimo, v_cont, v_estoque_min_pc.

  IF t_saida-wrk02 IS NOT INITIAL.
    v_werks = t_saida-wrk02.
  ELSE.
    v_werks = t_saida-werks.
  ENDIF.

  SELECT matnr stlnr UP TO 1 ROWS
    INTO (mast-matnr, mast-stlnr)
    FROM mast
    WHERE matnr = t_saida-matnr
    AND   werks = v_werks.
  ENDSELECT.

  CHECK mast-stlnr IS NOT INITIAL.

  SELECT stlty stlnr stlkn idnrk meins menge
    INTO TABLE t_stpo_aux
    FROM stpo
    WHERE stlnr = mast-stlnr.

  CHECK sy-subrc IS INITIAL.

  SELECT *
    INTO TABLE t_makt
    FROM makt
    FOR ALL ENTRIES IN t_stpo_aux
    WHERE matnr = t_stpo_aux-idnrk.

  APPEND LINES OF t_stpo_aux TO t_stpo.

  v_prev_org       = t_saida-prev_org.
  v_estoque_minimo = t_saida-estoque_minimo.
  v_estoque_min_pc = t_saida-est_min_pec.

  LOOP AT t_stpo WHERE stlnr = mast-stlnr.

    v_tabix = sy-tabix.

    IF t_stpo-idnrk NOT BETWEEN '54000000' AND '54999999'.
      CONTINUE.
    ENDIF.

    CLEAR: wa_saida, v_roteiro, v_cuobj, v_atflv.

    READ TABLE t_makt WITH KEY matnr = t_stpo-idnrk.
    IF sy-subrc IS INITIAL.
      wa_saida-descricao = t_makt-maktx.
    ENDIF.

    wa_saida-werks = v_werks.
    wa_saida-matnr     = t_stpo-idnrk.
    wa_saida-cod_cli   = t_zrnppt_prev-kunnr.
    wa_saida-cliente   = t_saida-cliente.
    wa_saida-codigo    = t_saida-codigo.
***Inicio - Fabio - 25.01.2021 - RDM 251520
    wa_saida-tp_cliente    = t_saida-tp_cliente.
***Fim - Fabio - 25.01.2021 - RDM 251520


    SELECT cuobj UP TO 1 ROWS
      INTO v_cuobj
      FROM inob
      WHERE obtab = 'MARA'
        AND objek = t_stpo-idnrk
        AND klart = '023'.
    ENDSELECT.

    IF sy-subrc IS INITIAL.
      SELECT atflv UP TO 1 ROWS
        INTO v_atflv
        FROM ausp
        WHERE objek = v_cuobj
          AND atinn = v_atinn_sd_peso_blank
          AND klart = '023'.
      ENDSELECT.

      IF sy-subrc IS INITIAL AND v_atflv IS NOT INITIAL.
        v_converte         = v_atflv.
        wa_saida-peso_peca = v_converte.
      ENDIF.
    ENDIF.

    wa_saida-est_min_pec = v_estoque_min_pc.
    IF t_stpo-meins = 'KG'.
*INC 3699045
      IF v_qtde = 0.
        v_qtde = 1.
      ENDIF.
*INC 3699045
      wa_saida-prev_org = v_prev_org * t_stpo-menge / ( v_qtde * 1000 ).
      wa_saida-estoque_minimo = ( v_estoque_min_pc * t_stpo-menge ) / 1000 .
    ELSE.
      IF v_qtde = 0.
        v_qtde = 1.
      ENDIF.
      wa_saida-prev_org = v_prev_org * t_stpo-menge / v_qtde.
      wa_saida-estoque_minimo = ( v_estoque_min_pc * t_stpo-menge ).
    ENDIF.

    PERFORM roteiro_mat_configurado USING t_stpo-idnrk
                                          v_werks.

    wa_saida-roteiro = v_roteiro.
    wa_saida-ferramenta = v_ferramenta.

    CLEAR: t_saida.
    MOVE wa_saida TO t_saida.
    APPEND t_saida.

    t_zrnppt_prev_aux2 = t_zrnppt_prev.
    MOVE-CORRESPONDING wa_saida TO t_zrnppt_prev_aux2.
    t_zrnppt_prev_aux2-kunnr = wa_saida-cod_cli.
    t_zrnppt_prev_aux2-bismt = wa_saida-cliente.
    APPEND t_zrnppt_prev_aux2.

    t_stpo-matnr = wa_saida-matnr.
    MODIFY t_stpo INDEX v_tabix.

  ENDLOOP.

ENDFORM.                    " F_APPEND_MAT_INTERMEDIARIO
*INC 3097794

*INC 3097794
* Bruno Quadros - CSC - 30.09.2010 - Inicio
*&---------------------------------------------------------------------*
*&      Form  F_APPEND_MAT_PRIMA
*&---------------------------------------------------------------------*
*FORM f_append_mat_prima .
*
*  DATA: t_makt TYPE TABLE OF makt WITH HEADER LINE,
*        v_prev_org TYPE zrnppt_prev-peso,
*        v_estoque_minimo LIKE zrnppt_prev-estoque_minimo.
*****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Início
*  DATA: v_estoque_min_pc LIKE zrnppt_prev-estoque_min_pc.
*****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Fim
** Bruno Quadros - CSC - 29.07.2011 - Inicio
*  DATA: v_atflv TYPE atflv,
*        v_cont TYPE i,
*        v_cuobj LIKE inob-cuobj,
*        v_converte TYPE p DECIMALS 2,
*        v_idnrk TYPE zqmt_partn_mrp-idnrk,
*        v_werks TYPE aufk-werks.
*
*  DATA t_stpo_aux LIKE TABLE OF t_stpo WITH HEADER LINE.
** Bruno Quadros - CSC - 29.07.2011 - Fim
*
*  CLEAR: wa_saida, mast-stlnr, t_stpo, t_makt[], v_prev_org, v_estoque_minimo, v_cont, v_estoque_min_pc.
*
** Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
*  IF t_saida-wrk02 IS NOT INITIAL.
*    v_werks = t_saida-wrk02.
*  ELSE.
****Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
**    v_werks = p_centro-low.
*    v_werks = t_saida-werks.
****Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
*  ENDIF.
** Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Fim
*
*  SELECT matnr stlnr UP TO 1 ROWS
*    INTO (mast-matnr, mast-stlnr)
*    FROM mast
*    WHERE matnr = t_saida-matnr
** Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Inicio
*    AND   werks = v_werks.
** Inclusao - Sidney Q. de Andrade - SU - 03.01.2012 - Fim
*  ENDSELECT.
*
*  CHECK mast-stlnr IS NOT INITIAL.
*
*  SELECT stlty stlnr stlkn idnrk meins menge
*    INTO TABLE t_stpo_aux
*    FROM stpo
*    WHERE stlnr = mast-stlnr.
*
*  CHECK sy-subrc IS INITIAL.
*
*  v_cont = sy-dbcnt.
*
*  SELECT *
*    INTO TABLE t_makt
*    FROM makt
*    FOR ALL ENTRIES IN t_stpo_aux
*    WHERE matnr = t_stpo_aux-idnrk.
*
*  APPEND LINES OF t_stpo_aux TO t_stpo.
*
*  IF t_saida-descricao(3) = 'BKS' OR
**RDM 240520 - INICIO
*     t_saida-descricao(3) = 'PLT'.
**RDM 240520 - FIM
*
*    v_prev_org       = t_saida-prev_org.
*    v_estoque_minimo = t_saida-estoque_minimo.
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Início
*    v_estoque_min_pc = t_saida-est_min_pec.
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Fim
*
*    LOOP AT t_stpo WHERE stlnr = mast-stlnr.
*
*      v_tabix = sy-tabix.
*
**RDM 240520
*      IF t_stpo-idnrk NOT BETWEEN '54000000' AND '54999999'.
*        CONTINUE.
*      ENDIF.
**RDM 240520
*
*      CLEAR: wa_saida, v_roteiro, v_cuobj, v_atflv.
*
*      READ TABLE t_makt WITH KEY matnr = t_stpo-idnrk.
*      IF sy-subrc IS INITIAL.
*        wa_saida-descricao = t_makt-maktx.
*      ENDIF.
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Início
*      wa_saida-werks = v_werks.
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Fim
*      wa_saida-matnr     = t_stpo-idnrk.
*      wa_saida-cod_cli   = t_zrnppt_prev-kunnr.
*      wa_saida-cliente   = t_saida-cliente.
*      wa_saida-codigo    = t_saida-codigo.
*
**   Bruno Quadros - CSC - 29.07.2011 - Inicio
**      CALL FUNCTION 'ZRNMMF_SEARCH_CMAT'
**        EXPORTING
**          matnr                     = t_stpo-idnrk
**          klart                     = '023'
**          idioma                    = sy-langu
**          caracteristica            = v_atinn_sd_peso_blank
**        TABLES
**          itab_mat                  = t_caract
**        EXCEPTIONS
**          matnr_not_found           = 1
**          matnr_or_mara_not_found   = 2
**          caracteristicas_not_found = 3
**          classe_not_found          = 4
**          OTHERS                    = 5.
*
*      SELECT cuobj UP TO 1 ROWS
*        INTO v_cuobj
*        FROM inob
*        WHERE obtab = 'MARA'
*          AND objek = t_stpo-idnrk
*          AND klart = '023'.
*      ENDSELECT.
*
*      IF sy-subrc IS INITIAL.
*        SELECT atflv UP TO 1 ROWS
*          INTO v_atflv
*          FROM ausp
*          WHERE objek = v_cuobj
*            AND atinn = v_atinn_sd_peso_blank
*            AND klart = '023'.
*        ENDSELECT.
*
*        IF sy-subrc IS INITIAL AND v_atflv IS NOT INITIAL.
*          v_converte         = v_atflv.
*          wa_saida-peso_peca = v_converte.
*        ENDIF.
*      ENDIF.
**   Bruno Quadros - CSC - 29.07.2011 - Fim
*
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Início
*      wa_saida-est_min_pec = v_estoque_min_pc.
*      IF t_stpo-meins = 'KG'.
*        wa_saida-prev_org = v_prev_org * t_stpo-menge / ( v_qtde * 1000 ).
**        wa_saida-estoque_minimo = v_estoque_minimo * t_stpo-menge / ( v_qtde * 1000 ).
*        wa_saida-estoque_minimo = ( v_estoque_min_pc * t_stpo-menge ) / 1000 .
*      ELSE.
**RDM 240520
*        IF v_qtde = 0.
*          v_qtde = 1.
*        ENDIF.
**RDM 240520
*        wa_saida-prev_org = v_prev_org * t_stpo-menge / v_qtde.
**        wa_saida-estoque_minimo = v_estoque_minimo * t_stpo-menge / v_qtde.
*        wa_saida-estoque_minimo = ( v_estoque_min_pc * t_stpo-menge ).
*      ENDIF.
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Fim
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Início
*      PERFORM roteiro_mat_configurado USING t_stpo-idnrk
**                                            t_zrnppt_prev-werks.
*                                            v_werks.
****Inserção - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Fim
*      wa_saida-roteiro = v_roteiro.
**   Bruno Quadros - CSC - 06.04.2011 - Inicio
*      wa_saida-ferramenta = v_ferramenta.
**   Bruno Quadros - CSC - 06.04.2011 - Fim
*
*      CLEAR: t_saida.
*      MOVE wa_saida TO t_saida.
*      APPEND t_saida.
*
*      t_zrnppt_prev_aux2 = t_zrnppt_prev.
*      MOVE-CORRESPONDING wa_saida TO t_zrnppt_prev_aux2.
*      t_zrnppt_prev_aux2-kunnr = wa_saida-cod_cli.
*      t_zrnppt_prev_aux2-bismt = wa_saida-cliente.
*      APPEND t_zrnppt_prev_aux2.
*
** Bruno Quadros - CSC - 29.07.2011 - Inicio
*      t_stpo-matnr = mast-matnr.
*      MODIFY t_stpo INDEX v_tabix.
** Bruno Quadros - CSC - 29.07.2011 - Fim
*    ENDLOOP.
*
*  ELSE.
*
*    IF t_saida-strgr = '25'.
*
*      DATA: lv_atinn LIKE cabn-atinn,
*            lv_partnumber TYPE zqmt_partnumber-partnumber,
*            v_resul TYPE zqmt_partnumber-resul.
**            t_partnumber TYPE TABLE OF zqmt_partnumber WITH HEADER LINE.
****Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Início
**      CALL FUNCTION 'CONVERSION_EXIT_ATINN_INPUT'
**        EXPORTING
**          input  = 'SD_NUM_MAT1'
**        IMPORTING
**          output = lv_atinn.
**
**      SELECT SINGLE resul
**         FROM zqmt_partnumber
**         INTO v_resul
**         WHERE kdmat  = t_saida-codigo AND
**               atnam  = lv_atinn  AND
**             ( status = 'LIB' OR status = 'LIR' ).
****Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Início
**      CLEAR lv_partnumber.
**      SELECT SINGLE partnumber
**        FROM zqmt_partnumber
**        INTO lv_partnumber
*****Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
***       WHERE kdmat = t_saida-pn AND
**        WHERE matnr = t_saida-matnr   AND
**              kunnr = t_saida-cod_cli AND
**              kdmat = t_saida-codigo AND
**        ( datefrom <= p_period-low  AND dateto >= p_period-high ) AND
*****Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
**              atnam = 'SD_MATERIAL_CLIENTE' AND
**              ( status   = 'LIB' OR status = 'LIR' ).
****Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Fim
*      CLEAR: v_idnrk.
*      SELECT SINGLE idnrk
*        INTO v_idnrk
*        FROM zqmt_partn_mrp
****Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Início
**       WHERE partnumber = lv_partnumber
*        WHERE partnumber =  t_saida-pn_interno
****Inserção - Hermiro Carvalho - PRJ01313 - 21/10/2015 - Fim
*         AND werks = t_saida-werks.
*
****Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Fim
*      IF sy-subrc IS INITIAL.
*        t_saida-materia_prima = v_idnrk.
*
*        LOOP AT t_stpo WHERE stlnr = mast-stlnr.
*          t_stpo-matnr = t_saida-matnr.
*          t_stpo-idnrk = t_saida-materia_prima.
*          MODIFY t_stpo INDEX sy-tabix.
*        ENDLOOP.
*
*      ENDIF.
*
*    ELSE.
*
** Bruno Quadros - CSC - 29.07.2011 - Inicio
*      LOOP AT t_stpo WHERE stlnr = mast-stlnr.
*        t_stpo-matnr = t_saida-matnr.
*        MODIFY t_stpo INDEX sy-tabix.
*      ENDLOOP.
** Bruno Quadros - CSC - 29.07.2011 - Fim
*      IF sy-subrc IS INITIAL.
*        IF v_cont = 1.
*          t_saida-materia_prima = t_stpo-idnrk.
*        ELSE.
*          t_saida-materia_prima = '*'.
*        ENDIF.
*      ENDIF.
*
*    ENDIF.
*
*  ENDIF.
*
*ENDFORM.                    " F_APPEND_MAT_PRIMA
* Bruno Quadros - CSC - 30.09.2010 - Fim
*INC 3097794

*INC 3097794
*&---------------------------------------------------------------------*
*&      Form  F_APPEND_MAT_PRIMA
*&---------------------------------------------------------------------*
FORM f_append_mat_prima .

  DATA t_stpo_aux LIKE TABLE OF t_stpo WITH HEADER LINE.

  DATA: v_cont  TYPE i,
        v_werks TYPE aufk-werks.

  CLEAR: mast-stlnr, t_stpo, v_cont, v_werks.

  IF t_saida-wrk02 IS NOT INITIAL.
    v_werks = t_saida-wrk02.
  ELSE.
    v_werks = t_saida-werks.
  ENDIF.

  SELECT matnr stlnr UP TO 1 ROWS
    INTO (mast-matnr, mast-stlnr)
    FROM mast
    WHERE matnr = t_saida-matnr
    AND   werks = v_werks.
  ENDSELECT.

  CHECK mast-stlnr IS NOT INITIAL.

  SELECT stlty stlnr stlkn idnrk meins menge
    INTO TABLE t_stpo_aux
    FROM stpo
    WHERE stlnr = mast-stlnr.

  CHECK sy-subrc IS INITIAL.

  v_cont = sy-dbcnt.

  APPEND LINES OF t_stpo_aux TO t_stpo.

  LOOP AT t_stpo WHERE stlnr = mast-stlnr.
    t_stpo-matnr = t_saida-matnr.
    MODIFY t_stpo INDEX sy-tabix.
  ENDLOOP.

  IF v_cont = 1.
    t_saida-materia_prima = t_stpo-idnrk.
  ELSE.
    t_saida-materia_prima = '*'.
  ENDIF.

ENDFORM.                    " F_APPEND_MAT_PRIMA
*INC 3097794

*&---------------------------------------------------------------------*
*&      Form  AJUSTA_TABELA_SAIDA
*&---------------------------------------------------------------------*
*& Perform responsável pelo layout Sintético
*&---------------------------------------------------------------------*
FORM ajusta_tabela_saida.

  DATA: t_saida_aux  LIKE TABLE OF t_saida WITH HEADER LINE,
        t_saida_aux2 LIKE TABLE OF t_saida WITH HEADER LINE.

  DATA: wa_color_aux TYPE lvc_s_scol.
  DATA: vl_cobfate TYPE i, "Cobertura Fat+expedicao ORG
        vl_cobexp  TYPE i, "Cobertura expedicao ORG
        vl_cobfat  TYPE i, "Cobertura Faturado ORG
        vl_cobopt  TYPE i, "Cobertura OP Total Demanda ORG
        vl_cobmpt  TYPE i, "Cobertura MP em Fornecedor Demanda ORG
        vl_cobpco  TYPE i, "Cobertura de Pedidos de compra ORG
        vl_cobe60  TYPE i, "Cobertura expedicao (60 dias)
        vl_cobfat6 TYPE i. "Cobertura Fat+expedicao ORG (60 dias)

  IF t_saida[] IS NOT INITIAL.

    SORT t_saida BY werks lgort cliente cod_cli.
    CLEAR: t_saida_aux[], t_saida_aux2[], t_saida_aux, t_saida_aux2,
           vl_cobfate, vl_cobexp, vl_cobfat, vl_cobopt, vl_cobmpt,
           vl_cobpco, vl_cobe60, vl_cobfat6, wa_color_aux.
    t_saida_aux[] = t_saida[].
    CLEAR: t_saida[].
    LOOP AT t_saida_aux.
      t_saida_aux2[] = t_saida_aux[].
      DELETE t_saida_aux2 WHERE werks     NE t_saida_aux-werks
                             OR lgort     NE t_saida_aux-lgort
                             OR cliente   NE t_saida_aux-cliente
                             OR cod_cli   NE t_saida_aux-cod_cli
                             OR matnr     NE t_saida_aux-matnr.
      LOOP AT t_saida_aux2."Contabiliza os dados depois que a tabela foi deletada
        PERFORM calculo_cobertura USING t_saida_aux2 wa_color_aux CHANGING wa_saida wa_saida-cellcolor.
      ENDLOOP.
      wa_saida-werks    = t_saida_aux2-werks.
      wa_saida-matnr    = t_saida_aux2-matnr.
      wa_saida-lgort    = t_saida_aux2-lgort.
      wa_saida-cliente  = t_saida_aux2-cliente.
      wa_saida-cod_cli  = t_saida_aux2-cod_cli.
      wa_saida-materia_prima  = t_saida_aux2-materia_prima.
***Inserção - Hermiro Carvalho - RDM 196641 - 21/08/2018 - Início
***S/MP
      t_saida_aux2-s_mp  = t_saida_aux2-estoque_mp + t_saida_aux2-estoque_mp_cen + t_saida_aux2-estoque_ltan +
                           t_saida_aux2-estoque_forn + t_saida_aux2-estoque_forn_ltan.
      IF t_saida_aux2-s_mp > 3.
        wa_saida-s_mp = 0.
      ELSE.
        wa_saida-s_mp = 1.
      ENDIF.
***Inserção - Hermiro Carvalho - RDM 196641 - 21/08/2018 - Fim
      APPEND wa_saida TO t_saida." FROM wa_saida INDEX vl_index. "TESTE RDM 196641
      DELETE t_saida_aux WHERE werks    EQ t_saida_aux-werks
                           AND lgort    EQ t_saida_aux-lgort
                           AND cliente  EQ t_saida_aux-cliente
                           AND cod_cli  EQ t_saida_aux-cod_cli
                           AND matnr    EQ t_saida_aux-matnr.
      CLEAR: wa_saida, t_saida_aux2[], t_saida_aux2, wa_color_aux, t_saida_aux.
    ENDLOOP.
  ENDIF.

ENDFORM.                    " AJUSTA_TABELA_SAIDA

* Bruno Quadros - CSC - 12.04.2011 - Inicio
*&---------------------------------------------------------------------*
*&      Form  ZF_BUSCA_VARIANTE
*&---------------------------------------------------------------------*
FORM zf_busca_variante .

  CALL FUNCTION 'REUSE_ALV_VARIANT_F4'
    EXPORTING
      is_variant    = gs_variante
      i_save        = 'U'
    IMPORTING
      es_variant    = gs_defvariante
    EXCEPTIONS
      not_found     = 1
      program_error = 2
      OTHERS        = 3.
  IF sy-subrc <> 0.
    MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
            WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
  ELSE.
    pc_vari = gs_defvariante-variant.
  ENDIF.

ENDFORM.                    " ZF_BUSCA_VARIANTE

*&---------------------------------------------------------------------*
*&      Form  ZF_VALIDA_VARIANTE
*&---------------------------------------------------------------------*
FORM zf_valida_variante .

  IF NOT pc_vari IS INITIAL.

    CALL FUNCTION 'REUSE_ALV_VARIANT_EXISTENCE'
      EXPORTING
        i_save        = 'U'
      CHANGING
        cs_variant    = gs_defvariante
      EXCEPTIONS
        wrong_input   = 1
        not_found     = 2
        program_error = 3
        OTHERS        = 4.

    gs_variante = gs_defvariante.
  ELSE.
    CLEAR gs_variante.
    gs_variante-report = sy-repid.
  ENDIF.

ENDFORM.                    " ZF_VALIDA_VARIANTE

*&---------------------------------------------------------------------*
*&      Form  ZF_INICIALIZA
*&---------------------------------------------------------------------*
FORM zf_inicializa .

  CLEAR gs_variante.
  gs_variante-report   = sy-repid.
  gs_variante-username = sy-uname.
  gs_defvariante       = gs_variante.

  CALL FUNCTION 'REUSE_ALV_VARIANT_DEFAULT_GET'
    EXPORTING
      i_save        = 'U'
    CHANGING
      cs_variant    = gs_defvariante
    EXCEPTIONS
      wrong_input   = 1
      not_found     = 2
      program_error = 3
      OTHERS        = 4.

  IF sy-subrc <> 0.
    pc_vari = gs_defvariante-variant.
  ENDIF.

ENDFORM.                    " ZF_INICIALIZA
* Bruno Quadros - CSC - 12.04.2011 - Fim

*&---------------------------------------------------------------------*
*&      Form  F_INICIA_RANGE
*&---------------------------------------------------------------------*
FORM f_inicia_range .

  FREE r_auart.
  CLEAR r_auart.
  r_auart-sign   = 'I'.
  r_auart-option = 'EQ'.
  r_auart-low    = 'PP01'.
  CLEAR r_auart-high.
  APPEND r_auart.

  r_auart-sign   = 'I'.
  r_auart-option = 'EQ'.
  r_auart-low    = 'PP05'.
  CLEAR r_auart-high.
  APPEND r_auart.

  r_auart-sign   = 'I'.
  r_auart-option = 'EQ'.
  r_auart-low    = 'PP02'.
  CLEAR r_auart-high.
  APPEND r_auart.

  PERFORM atricara USING 'SD_PESO_BLANK' CHANGING v_atinn_sd_peso_blank.

ENDFORM.                    " F_INICIA_RANGE

*&---------------------------------------------------------------------*
*&      Form  F_AJUSTA_P_MAT
*&---------------------------------------------------------------------*
FORM f_ajusta_p_mat .

  IF r_jit = 'X'.

***Inserção - Sidney Q de Andrade - RDM 177856 - 20/03/17 - Início
    p_mat-sign   = 'I'.
    p_mat-option = 'BT'.
    p_mat-low    = '12000000'.
    p_mat-high   = '12999999'.
    APPEND p_mat.
***Inserção - Sidney Q de Andrade - RDM 177856 - 20/03/17 - Fim

    p_mat-sign   = 'I'.
    p_mat-option = 'BT'.
    p_mat-low    = '30000000'.
    p_mat-high   = '34999999'.
    APPEND p_mat.

  ELSEIF r_serv = 'X'.

    p_mat-sign   = 'I'.
    p_mat-option = 'BT'.
    p_mat-low    = '85000000'.
    p_mat-high   = '86999999'.
    APPEND p_mat.

  ELSEIF r_dist = 'X'.

***Inserção - Sidney Q de Andrade - RDM 177856 - 20/03/17 - Início
    p_mat-sign   = 'I'.
    p_mat-option = 'BT'.
    p_mat-low    = '12000000'.
    p_mat-high   = '12999999'.
    APPEND p_mat.
***Inserção - Sidney Q de Andrade - RDM 177856 - 20/03/17 - Fim

    p_mat-sign   = 'E'.
    p_mat-option = 'BT'.
    p_mat-low    = '34000000'.
    p_mat-high   = '34999999'.
    APPEND p_mat.

    p_mat-sign   = 'E'.
    p_mat-option = 'BT'.
    p_mat-low    = '85000000'.
    p_mat-high   = '86999999'.
    APPEND p_mat.

  ENDIF.

ENDFORM.                    " F_AJUSTA_P_MAT

*&---------------------------------------------------------------------*
*&      Form  CENTRO_PRODUTOR
*&---------------------------------------------------------------------*
FORM centro_produtor .

  DATA: v_convert TYPE p DECIMALS 6.

* Dados de centro para material
  DATA: BEGIN OF t_marc_aux OCCURS 0,
          matnr LIKE marc-matnr,
          werks LIKE marc-werks,
          sobsl LIKE marc-sobsl,
          wrk02 LIKE t460a-wrk02,       "centro produtor
        END OF t_marc_aux.

  "Verificar centro produtor
  SELECT m~matnr m~werks m~sobsl t~wrk02
    INTO TABLE t_marc_aux
    FROM marc AS m INNER JOIN t460a AS t
        ON ( m~werks = t~werks AND
             m~sobsl = t~sobsl )
    FOR ALL ENTRIES IN t_saida
    WHERE m~matnr = t_saida-matnr
***Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*      AND m~werks = p_centro-low
      AND m~werks = t_saida-werks
***Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
      AND m~sobsl <> ''.

***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/20147 - Início
*  SORT t_marc_aux BY matnr.
  SORT t_marc_aux BY matnr werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/20147 - Fim

* Bruno Quadros - CSC - 29.07.2011 - Inicio
  IF t_zrnppt_prev_aux2[] IS NOT INITIAL.
* Busca estratégia do material
    SELECT matnr werks strgr minbe bstma shzet lgpro sobsl
      APPENDING TABLE t_marc
      FROM marc
      FOR ALL ENTRIES IN t_zrnppt_prev_aux2
      WHERE matnr = t_zrnppt_prev_aux2-matnr
***Alteração - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Início
*      AND werks = p_centro-low
*      AND werks = t_saida-werks.
       AND werks = t_zrnppt_prev_aux2-werks.
***Alteração - Hermiro Carvalho - RDM 105472 - 15/08/2014 - Fim

    IF sy-subrc = 0.
***Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*      SORT t_marc BY matnr werks.
      SORT t_marc BY matnr.
***Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
      DELETE ADJACENT DUPLICATES FROM t_marc COMPARING matnr werks.
    ENDIF.
  ENDIF.
* Bruno Quadros - CSC - 29.07.2011 - Fim


  LOOP AT t_saida.
    v_tabix = sy-tabix.
    CLEAR t_marc_aux-wrk02.
    READ TABLE t_marc_aux WITH KEY matnr = t_saida-matnr
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Início
                                   werks = t_saida-werks
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Fim
                                   BINARY SEARCH.
    IF t_marc_aux-wrk02 IS NOT INITIAL.
      t_saida-wrk02 = t_marc_aux-wrk02.
    ELSE.
***Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*      t_saida-wrk02 = p_centro-low.
      t_saida-wrk02 = t_saida-werks.
***Alteração - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
    ENDIF.

* Bruno Quadros - CSC - 29.07.2011 - Inicio

*INC 3097794
*    IF t_saida-descricao(3) NE 'BKS' AND
*       t_saida-descricao(3) NE 'PLT'.
*      PERFORM f_append_mat_prima.
*    ENDIF.
    PERFORM f_append_mat_prima.
*INC 3097794


    IF t_saida-peso_peca IS NOT INITIAL.
      v_convert = t_saida-peso_peca / 1000.
      READ TABLE t_mara WITH KEY matnr = t_saida-matnr
                                 BINARY SEARCH.
      IF sy-subrc IS INITIAL.
        READ TABLE t_marc WITH KEY matnr = t_saida-matnr
                                   werks = t_saida-werks
                                   BINARY SEARCH.
        IF sy-subrc IS INITIAL AND t_marc-bstma IS NOT INITIAL.

          IF t_mara-meins = 'PC' OR t_mara-meins = 'ST'.
            t_saida-max_expe_pc  = t_marc-bstma.
            t_saida-max_expe_ton = v_convert * t_saida-max_expe_pc.
          ELSEIF t_mara-meins = 'TO' OR t_mara-meins = 'TN'.
            t_saida-max_expe_pc  = t_marc-bstma / v_convert.
            t_saida-max_expe_ton = t_marc-bstma.
          ENDIF.

        ENDIF.
      ENDIF.
    ENDIF.
* Bruno Quadros - CSC - 29.07.2011 - Fim

***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Início
    CLEAR: v_ok, v_ferramenta.
    PERFORM check_mat_configurado  USING t_saida-matnr
                                         t_saida-werks
                                         CHANGING v_ok.
    IF NOT v_ferramenta IS INITIAL.
      t_saida-ferramenta = v_ferramenta.
    ENDIF.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim

* Início da Alteração - Nelson Junior - Stefanini - 18.08.2025
*    PERFORM: zf_qualifica_demanda.
* Fim da Alteração - Nelson Junior - Stefanini - 18.08.2025

    MODIFY t_saida INDEX v_tabix.
  ENDLOOP.

* INICIO: 09mar2017 - RGC8096
  IF t_saida[] IS NOT INITIAL.

    LOOP AT t_saida.
      CLEAR: wa_melhoria.
      wa_melhoria-werks      = t_saida-werks.
      wa_melhoria-matnr      = t_saida-matnr.
      wa_melhoria-wrk02      = t_saida-wrk02. " 09mar2017 - RGC8096 - RDM 176853
      APPEND wa_melhoria TO t_melhoria.
    ENDLOOP.

    SORT t_melhoria BY werks matnr.
    DELETE ADJACENT DUPLICATES FROM t_melhoria COMPARING werks matnr.

  ENDIF.
* FIM: 09mar2017 - RGM8096

ENDFORM.                    " CENTRO_PRODUTOR

*&---------------------------------------------------------------------*
*&      Form  AJUSTA_TAB_EXPEDICAO
*&---------------------------------------------------------------------*
FORM ajusta_tab_i USING p_value.

  DATA: wa_pl_corte	TYPE zrnppt_pl_corte,
        wa_lfa1     TYPE lfa1.
  DATA: vl_strgr  TYPE marc-strgr.
  READ TABLE t_saida INDEX v_tabix.

  DATA: t_zpetiq  TYPE TABLE OF zrnppt_ov_separa,
        wa_zpetiq TYPE zrnppt_ov_separa.

  DATA: t_mchb_aux LIKE TABLE OF t_mchb WITH HEADER LINE.

  DATA: lc_entrada TYPE ekbe-menge.

  IF p_value = 'T_SAIDA-EXPEDICAO' OR
     p_value = 'T_SAIDA-EXP_INTER' OR
     p_value = 'T_SAIDA-MAT_CQ'    OR
     p_value = 'T_SAIDA-EXPEDI_CEN'.


    CLEAR: t_zpetiq[], wa_zpetiq.

    t_mchb_aux[] = t_mchb[].
    SORT t_mchb_aux BY matnr.
    DELETE t_mchb_aux WHERE matnr <> t_saida-matnr.

    IF t_mchb_aux[] IS NOT INITIAL.

      SELECT matnr werks lgort charg vbeln posnr sequencia edatu
        FROM zrnppt_ov_separa
        INTO CORRESPONDING FIELDS OF TABLE t_zpetiq
        FOR ALL ENTRIES IN t_mchb_aux
        WHERE matnr  = t_mchb_aux-matnr
          AND werks  = t_mchb_aux-werks
          AND lgort  = t_mchb_aux-lgort
          AND charg  = t_mchb_aux-charg
          AND status <> 'C'.  "Etiqueta com Status = Cancelada

    ENDIF.

    READ TABLE t_marc WITH KEY matnr = t_saida-matnr
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Início
                               werks = t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Fim
    IF sy-subrc = 0.
      IF t_marc-strgr NE '25'
* Inclusão - Rodrigo Soeiro - 12/2011 - Inicio
     AND t_marc-strgr NE '20' .
* Fim da Inclusão

        LOOP AT t_mchb WHERE matnr = t_saida-matnr.

          CLEAR t_saida_i.

          IF t_mchb-clabs IS NOT INITIAL.

            IF t_saida-matnr GE '50000000' AND t_saida-matnr LE '54999999' AND p_value = 'T_SAIDA-EXP_INTER'.
              t_saida_i-peso    = t_mchb-clabs.
              t_saida_i-dt_prod = t_mchb-zdatreg.
            ELSEIF p_value = 'T_SAIDA-EXPEDICAO' AND t_mchb-werks = t_saida-werks.
              t_saida_i-peso    = t_mchb-clabs.
              t_saida_i-dt_prod = t_mchb-zdatreg.
            ELSEIF p_value = 'T_SAIDA-EXPEDI_CEN' AND t_mchb-werks NE t_saida-werks.
              t_saida_i-peso    = t_mchb-clabs.
              t_saida_i-dt_prod = t_mchb-zdatreg.
            ENDIF.

          ELSEIF t_mchb-cinsm IS NOT INITIAL AND p_value = 'T_SAIDA-MAT_CQ'.

            t_saida_i-peso = t_mchb-cinsm.

          ENDIF.

          READ TABLE t_zpetiq INTO wa_zpetiq WITH KEY matnr = t_mchb-matnr
                                                      werks = t_mchb-werks
                                                      lgort = t_mchb-lgort
                                                      charg = t_mchb-charg.
          IF sy-subrc IS INITIAL.
            t_saida_i-dt_zpetiq = wa_zpetiq-edatu.
          ENDIF.

          IF t_saida_i-peso IS NOT INITIAL.

            IF t_mchb-ov IS NOT INITIAL.
              SHIFT t_mchb-ov LEFT DELETING LEADING '0'.
              SHIFT t_mchb-it LEFT DELETING LEADING '0'.
              CONCATENATE t_mchb-ov '/' t_mchb-it INTO t_saida_i-ov_it.
            ENDIF.

            t_saida_i-znumpp      = t_mchb-znumpp.
            t_saida_i-werks       = t_mchb-werks.
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Início
*            t_saida_i-peca        = t_mchb-peca * 1000.
            t_saida_i-peca        = t_mchb-peca.
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Fim
            t_saida_i-matnr_charg = t_mchb-matnr.
            t_saida_i-charg       = t_mchb-charg.

            APPEND t_saida_i.

          ENDIF.

        ENDLOOP.

      ELSE.

        LOOP AT t_mchb WHERE matnr      = t_saida-matnr
                         AND matcli     = t_saida-codigo
                         AND ferramenta = t_saida-ferramenta.
          CLEAR t_saida_i.

          IF t_mchb-clabs IS NOT INITIAL.

            IF p_value = 'T_SAIDA-EXPEDICAO' AND t_mchb-werks = t_saida-werks.
              t_saida_i-peso    = t_mchb-clabs.
              t_saida_i-dt_prod = t_mchb-zdatreg.
            ELSEIF p_value = 'T_SAIDA-EXPEDI_CEN' AND t_mchb-werks NE t_saida-werks.
              t_saida_i-peso    = t_mchb-clabs.
              t_saida_i-dt_prod = t_mchb-zdatreg.
            ENDIF.

          ELSEIF t_mchb-cinsm IS NOT INITIAL AND p_value = 'T_SAIDA-MAT_CQ'.

            t_saida_i-peso = t_mchb-cinsm.

          ENDIF.

          READ TABLE t_zpetiq INTO wa_zpetiq WITH KEY matnr = t_mchb-matnr
                                                      werks = t_mchb-werks
                                                      lgort = t_mchb-lgort
                                                      charg = t_mchb-charg.
          IF sy-subrc IS INITIAL.
            t_saida_i-dt_zpetiq = wa_zpetiq-edatu.
          ENDIF.

          IF t_saida_i-peso IS NOT INITIAL.

            IF t_mchb-ov IS NOT INITIAL.
              SHIFT t_mchb-ov LEFT DELETING LEADING '0'.
              SHIFT t_mchb-it LEFT DELETING LEADING '0'.
              CONCATENATE t_mchb-ov '/' t_mchb-it INTO t_saida_i-ov_it.
            ENDIF.

            t_saida_i-znumpp      = t_mchb-znumpp.
            t_saida_i-werks       = t_mchb-werks.
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Início
*            t_saida_i-peca        = t_mchb-peca * 1000.
            t_saida_i-peca        = t_mchb-peca.
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Fim
            t_saida_i-matnr_charg = t_mchb-matnr.
            t_saida_i-charg       = t_mchb-charg.

            APPEND t_saida_i.

          ENDIF.

        ENDLOOP.

      ENDIF.

    ENDIF.

  ELSEIF p_value = 'T_SAIDA-FATURADO1'.

    READ TABLE t_marc WITH KEY matnr = t_saida-matnr
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Início
                               werks = t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Fim
    IF sy-subrc IS INITIAL.
      IF t_marc-strgr = '25'
* Inclusão - Rodrigo Soeiro - 12/2011 - Inicio
      OR t_marc-strgr EQ '20' .
* Fim da Inclusão

        LOOP AT t_vbrp WHERE matnr  = t_saida-matnr
                         AND matcli = t_saida-codigo
                         AND kunag  = t_saida-cod_cli
                         AND werks  = t_saida-werks.

          t_saida_i-werks = t_vbrp-werks.
          t_saida_i-nf    = t_vbrp-xblnr.
          SHIFT t_saida_i-nf LEFT DELETING LEADING '0'.

          IF t_vbrp-vbeln IS NOT INITIAL.
            SHIFT t_vbrp-vbeln LEFT DELETING LEADING '0'.
            SHIFT t_vbrp-posnr LEFT DELETING LEADING '0'.
            CONCATENATE t_vbrp-vbeln '/' t_vbrp-posnr INTO t_saida_i-faturamento.
          ENDIF.

* Bruno Quadros - CSC - 29.07.2011 - Inicio
          t_saida_i-dt_fatu = t_vbrp-erdat.
* Bruno Quadros - CSC - 29.07.2011 - Fim

          IF t_vbrp-aubel IS NOT INITIAL.
            SHIFT t_vbrp-aubel LEFT DELETING LEADING '0'.
            SHIFT t_vbrp-aupos LEFT DELETING LEADING '0'.
            CONCATENATE t_vbrp-aubel '/' t_vbrp-aupos INTO t_saida_i-ov_it.
          ENDIF.
          t_saida_i-peso = t_vbrp-ntgew.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*          t_saida_i-peca = t_vbrp-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida_i-peca = v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

          APPEND t_saida_i.
        ENDLOOP.

      ELSE.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Inicio
*        CLEAR: r_cod_cli, r_cod_cli[].
*
*        LOOP AT t_cod_cli WHERE sortl = t_saida-cliente.
*          r_cod_cli-sign   = 'I'.
*          r_cod_cli-option = 'EQ'.
*          r_cod_cli-low = t_cod_cli-kunnr.
*          APPEND r_cod_cli.
*          CLEAR r_cod_cli.
*        ENDLOOP.
*        LOOP AT t_vbrp WHERE matnr = t_saida-matnr
*                         AND kunag IN r_cod_cli
*                         AND matcli = t_saida-codigo
*                         AND werks  = t_saida-werks.

        LOOP AT t_vbrp WHERE matnr  = t_saida-matnr
                         AND matcli = t_saida-codigo
                         AND kunag  = t_saida-cod_cli
                         AND werks  = t_saida-werks.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Inicio
          t_saida_i-werks = t_vbrp-werks.
          t_saida_i-nf    = t_vbrp-xblnr.
          SHIFT t_saida_i-nf LEFT DELETING LEADING '0'.

          IF t_vbrp-vbeln IS NOT INITIAL.
            SHIFT t_vbrp-vbeln LEFT DELETING LEADING '0'.
            SHIFT t_vbrp-posnr LEFT DELETING LEADING '0'.
            CONCATENATE t_vbrp-vbeln '/' t_vbrp-posnr INTO t_saida_i-faturamento.
          ENDIF.

* Bruno Quadros - CSC - 29.07.2011 - Inicio
          t_saida_i-dt_fatu = t_vbrp-erdat.
* Bruno Quadros - CSC - 29.07.2011 - Fim

          IF t_vbrp-aubel IS NOT INITIAL.
            SHIFT t_vbrp-aubel LEFT DELETING LEADING '0'.
            SHIFT t_vbrp-aupos LEFT DELETING LEADING '0'.
            CONCATENATE t_vbrp-aubel '/' t_vbrp-aupos INTO t_saida_i-ov_it.
          ENDIF.
          t_saida_i-peso = t_vbrp-ntgew.

***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*          t_saida_i-peca = t_vbrp-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida_i-peca = v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

          APPEND t_saida_i.
        ENDLOOP.

      ENDIF.
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 158724 - 26/02/2016 - Início
    SORT: t_saida_i BY dt_fatu.
***Inserção - Hermiro Carvalho - RDM 158724 - 26/02/2016 - Fim
  ELSEIF p_value = 'T_SAIDA-FATURADO1_CEN'.

    READ TABLE t_marc WITH KEY matnr = t_saida-matnr
                               werks = t_saida-werks.

    IF sy-subrc IS INITIAL.
      IF t_marc-strgr = '25'
      OR t_marc-strgr EQ '20' .


        LOOP AT t_vbrp WHERE matnr  = t_saida-matnr
                         AND matcli = t_saida-codigo
                         AND kunag  = t_saida-cod_cli
                         AND werks  <> t_saida-werks.

          t_saida_i-werks = t_vbrp-werks.
          t_saida_i-nf    = t_vbrp-xblnr.
          SHIFT t_saida_i-nf LEFT DELETING LEADING '0'.

          IF t_vbrp-vbeln IS NOT INITIAL.
            SHIFT t_vbrp-vbeln LEFT DELETING LEADING '0'.
            SHIFT t_vbrp-posnr LEFT DELETING LEADING '0'.
            CONCATENATE t_vbrp-vbeln '/' t_vbrp-posnr INTO t_saida_i-faturamento.
          ENDIF.

* Bruno Quadros - CSC - 29.07.2011 - Inicio
          t_saida_i-dt_fatu = t_vbrp-erdat.
* Bruno Quadros - CSC - 29.07.2011 - Fim

          IF t_vbrp-aubel IS NOT INITIAL.
            SHIFT t_vbrp-aubel LEFT DELETING LEADING '0'.
            SHIFT t_vbrp-aupos LEFT DELETING LEADING '0'.
            CONCATENATE t_vbrp-aubel '/' t_vbrp-aupos INTO t_saida_i-ov_it.
          ENDIF.
          t_saida_i-peso = t_vbrp-ntgew.

***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*          t_saida_i-peca = t_vbrp-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida_i-peca = v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

          APPEND t_saida_i.
        ENDLOOP.

      ELSE.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Inicio
*        CLEAR: r_cod_cli, r_cod_cli[].
*
*        LOOP AT t_cod_cli WHERE sortl = t_saida-cliente.
*          r_cod_cli-sign   = 'I'.
*          r_cod_cli-option = 'EQ'.
*          r_cod_cli-low = t_cod_cli-kunnr.
*          APPEND r_cod_cli.
*          CLEAR r_cod_cli.
*        ENDLOOP.
*
*        LOOP AT t_vbrp WHERE matnr = t_saida-matnr
*                         AND kunag IN r_cod_cli
*                         AND matcli = t_saida-codigo
*                         AND werks  <> t_saida-werks.

        LOOP AT t_vbrp WHERE matnr  = t_saida-matnr
                         AND matcli = t_saida-codigo
                         AND kunag  = t_saida-cod_cli
                         AND werks  <> t_saida-werks.
***Inserção - Sidney Q de Andrade - INC 2170534 21/07/2016 - Fim
          t_saida_i-werks = t_vbrp-werks.
          t_saida_i-nf = t_vbrp-xblnr.
          SHIFT t_saida_i-nf LEFT DELETING LEADING '0'.

          IF t_vbrp-vbeln IS NOT INITIAL.
            SHIFT t_vbrp-vbeln LEFT DELETING LEADING '0'.
            SHIFT t_vbrp-posnr LEFT DELETING LEADING '0'.
            CONCATENATE t_vbrp-vbeln '/' t_vbrp-posnr INTO t_saida_i-faturamento.
          ENDIF.

          t_saida_i-dt_fatu = t_vbrp-erdat.

          IF t_vbrp-aubel IS NOT INITIAL.
            SHIFT t_vbrp-aubel LEFT DELETING LEADING '0'.
            SHIFT t_vbrp-aupos LEFT DELETING LEADING '0'.
            CONCATENATE t_vbrp-aubel '/' t_vbrp-aupos INTO t_saida_i-ov_it.
          ENDIF.
          t_saida_i-peso = t_vbrp-ntgew.

***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*          t_saida_i-peca = t_vbrp-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = t_vbrp-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = t_vbrp-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida_i-peca = v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

          APPEND t_saida_i.
        ENDLOOP.

      ENDIF.
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 158724 - 26/02/2016 - Início
    SORT: t_saida_i BY dt_fatu.
***Inserção - Hermiro Carvalho - RDM 158724 - 26/02/2016 - Fim
  ELSEIF p_value = 'T_SAIDA-FATURADO'.
    "Faturamento por Data de Remessa da OV

    LOOP AT it_vbep_vbap_vbak WHERE matnr = t_saida-matnr
                                AND kdmat = t_saida-codigo
                                AND kunnr = t_saida-cod_cli
                                AND werks = t_saida-werks.

      LOOP AT it_entped_it WHERE werks = it_vbep_vbap_vbak-werks AND
                                 vbeln = it_vbep_vbap_vbak-vbeln AND
                                 posnr = it_vbep_vbap_vbak-posnr AND
                                 etenr = it_vbep_vbap_vbak-etenr.

        CLEAR it_vbrp.
        READ TABLE it_vbrp WITH KEY vgbel = it_entped_it-vgbel
                                    aubel = it_entped_it-vbeln
                                    aupos = it_entped_it-posnr.

        t_saida_i-werks = it_vbrp-werks.
        t_saida_i-nf    = it_vbrp-xblnr.
        SHIFT t_saida_i-nf LEFT DELETING LEADING '0'.

        IF it_vbrp-vbeln IS NOT INITIAL.
          SHIFT it_vbrp-vbeln LEFT DELETING LEADING '0'.
          SHIFT it_vbrp-posnr LEFT DELETING LEADING '0'.
          CONCATENATE it_vbrp-vbeln '/' it_vbrp-posnr INTO t_saida_i-faturamento.
        ENDIF.

        t_saida_i-dt_fatu = it_vbrp-erdat.

        IF it_vbep_vbap_vbak-vbeln IS NOT INITIAL.
          SHIFT it_vbep_vbap_vbak-vbeln LEFT DELETING LEADING '0'.
          SHIFT it_vbep_vbap_vbak-posnr LEFT DELETING LEADING '0'.
          CONCATENATE it_vbep_vbap_vbak-vbeln '/' it_vbep_vbap_vbak-posnr INTO t_saida_i-ov_it.
        ENDIF.

        READ TABLE t_saida_i WITH KEY nf          = t_saida_i-nf
                                      ov_it       = t_saida_i-ov_it
                                      faturamento = t_saida_i-faturamento.
        IF sy-subrc IS NOT INITIAL.
          t_saida_i-peso = it_entped_it-peso_liq.

***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*          t_saida_i-peca = it_entped_it-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = it_entped_it-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = it_entped_it-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida_i-peca = v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

          APPEND t_saida_i.
        ELSE.
          t_saida_i-peso = t_saida_i-peso + it_entped_it-peso_liq.

***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*          t_saida_i-peca = t_saida_i-peca + it_entped_it-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = it_entped_it-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = it_entped_it-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida_i-peca = t_saida_i-peca + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

          MODIFY t_saida_i INDEX sy-tabix.
        ENDIF.

      ENDLOOP.
***Inserção - Hermiro Carvalho - RDM 158724 - 26/02/2016 - Início
      SORT: t_saida_i BY dt_fatu.
***Inserção - Hermiro Carvalho - RDM 158724 - 26/02/2016 - Fim
    ENDLOOP.

  ELSEIF p_value = 'T_SAIDA-FATURADO_CEN'.
    "Faturamento por Data de Remessa da OV

    LOOP AT it_vbep_vbap_vbak WHERE matnr = t_saida-matnr
                                AND kdmat = t_saida-codigo
                                AND kunnr = t_saida-cod_cli
                                AND werks <> t_saida-werks.

      LOOP AT it_entped_it WHERE werks = it_vbep_vbap_vbak-werks AND
                                 vbeln = it_vbep_vbap_vbak-vbeln AND
                                 posnr = it_vbep_vbap_vbak-posnr AND
                                 etenr = it_vbep_vbap_vbak-etenr.

        CLEAR it_vbrp.
        READ TABLE it_vbrp WITH KEY vgbel = it_entped_it-vgbel
                                    aubel = it_entped_it-vbeln
                                    aupos = it_entped_it-posnr.

        t_saida_i-werks = it_vbrp-werks.
        t_saida_i-nf    = it_vbrp-xblnr.
        SHIFT t_saida_i-nf LEFT DELETING LEADING '0'.

        IF it_vbrp-vbeln IS NOT INITIAL.
          SHIFT it_vbrp-vbeln LEFT DELETING LEADING '0'.
          SHIFT it_vbrp-posnr LEFT DELETING LEADING '0'.
          CONCATENATE it_vbrp-vbeln '/' it_vbrp-posnr INTO t_saida_i-faturamento.
        ENDIF.

        t_saida_i-dt_fatu = it_vbrp-erdat.

        IF it_vbep_vbap_vbak-vbeln IS NOT INITIAL.
          SHIFT it_vbep_vbap_vbak-vbeln LEFT DELETING LEADING '0'.
          SHIFT it_vbep_vbap_vbak-posnr LEFT DELETING LEADING '0'.
          CONCATENATE it_vbep_vbap_vbak-vbeln '/' it_vbep_vbap_vbak-posnr INTO t_saida_i-ov_it.
        ENDIF.

        READ TABLE t_saida_i WITH KEY nf          = t_saida_i-nf
                                      ov_it       = t_saida_i-ov_it
                                      faturamento = t_saida_i-faturamento.
        IF sy-subrc IS NOT INITIAL.
          t_saida_i-peso = it_entped_it-peso_liq.

***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*          t_saida_i-peca = it_entped_it-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = it_entped_it-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = it_entped_it-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida_i-peca = v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

          APPEND t_saida_i.
        ELSE.
          t_saida_i-peso = t_saida_i-peso + it_entped_it-peso_liq.

***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*          t_saida_i-peca = t_saida_i-peca + it_entped_it-fkimg.
          IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
            v_pecas = it_entped_it-fkimg.
          ELSE.
            IF t_saida-peso_peca IS NOT INITIAL.
              v_pecas = it_entped_it-fkimg / ( t_saida-peso_peca / 1000 ).
            ENDIF.
          ENDIF.
          t_saida_i-peca = t_saida_i-peca + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim

          MODIFY t_saida_i INDEX sy-tabix.
        ENDIF.

      ENDLOOP.

    ENDLOOP.

  ELSEIF p_value = 'T_SAIDA-OP_LIB'.

    READ TABLE t_saida INDEX v_tabix.

    LOOP AT t_dad_rel WHERE matnr  = t_saida-matnr
                        AND codigo = t_saida-codigo
                        AND sortl  = t_saida-cliente
                        AND op_lib IS NOT INITIAL.
      CLEAR t_saida_i.
      IF t_dad_rel-nr_ov IS NOT INITIAL.
        SHIFT t_dad_rel-nr_ov LEFT DELETING LEADING '0'.
        SHIFT t_dad_rel-it_ov LEFT DELETING LEADING '0'.
        CONCATENATE t_dad_rel-nr_ov '/' t_dad_rel-it_ov INTO t_saida_i-ov_it.
      ENDIF.
      t_saida_i-werks   = t_dad_rel-werks.
      t_saida_i-znumpp  = t_dad_rel-or_pro.
      t_saida_i-znumoa  = t_dad_rel-op.
      t_saida_i-peso    = t_dad_rel-op_lib.
      t_saida_i-peca    = t_dad_rel-op_peca.
      t_saida_i-wrk02   = t_dad_rel-werks2.

      CLEAR: t_proa[], t_proa, t_saida_i-roteiro.
      SELECT znumoa znumpp zoperpp zcentr
        INTO TABLE t_proa
        FROM zrnppt_proa
        WHERE znumoa = t_saida_i-znumoa
          AND znumpp = t_saida_i-znumpp
          AND ( zcvcont = 'PP01' OR zcvcont = 'PP02' OR zcvcont = 'ZP06' ).
      LOOP AT t_proa.
        IF t_saida_i-roteiro IS INITIAL.
          t_saida_i-roteiro = t_proa-zcentr.
        ELSE.
          CONCATENATE t_saida_i-roteiro t_proa-zcentr INTO t_saida_i-roteiro SEPARATED BY space.
        ENDIF.
      ENDLOOP.
*RDM322490 - GVCA20241122 - Begin
      SELECT SINGLE ernam erdat
        INTO (t_saida_i-ernam, t_saida_i-erdat)
        FROM aufk
        WHERE aufnr = t_saida_i-znumpp.
*RDM322490 - GVCA20241122 - End
      APPEND t_saida_i.
    ENDLOOP.

  ELSEIF p_value = 'T_SAIDA-OP_ABER'.

    READ TABLE t_saida INDEX v_tabix.

    LOOP AT t_dad_rel WHERE matnr  = t_saida-matnr
                        AND codigo = t_saida-codigo
                        AND sortl  = t_saida-cliente
                        AND op_aber IS NOT INITIAL.
      CLEAR t_saida_i.
      IF t_dad_rel-nr_ov IS NOT INITIAL.
        SHIFT t_dad_rel-nr_ov LEFT DELETING LEADING '0'.
        SHIFT t_dad_rel-it_ov LEFT DELETING LEADING '0'.
        CONCATENATE t_dad_rel-nr_ov '/' t_dad_rel-it_ov INTO t_saida_i-ov_it.
      ENDIF.
      t_saida_i-werks   = t_dad_rel-werks.
      t_saida_i-znumpp  = t_dad_rel-or_pro.
      t_saida_i-znumoa  = t_dad_rel-op.
      t_saida_i-peso    = t_dad_rel-op_aber.
      t_saida_i-peca    = t_dad_rel-op_peca.
***Inserção - Hermiro Carvalho - RDM 181010 - 30/05/2017 - Início
      t_saida_i-wrk02   = t_dad_rel-werks2.
***Inserção - Hermiro Carvalho - RDM 181010 - 30/05/2017 - Fim
      CLEAR: t_proa[], t_proa, t_saida_i-roteiro.
      SELECT znumoa znumpp zoperpp zcentr
        INTO TABLE t_proa
        FROM zrnppt_proa
        WHERE znumoa = t_saida_i-znumoa
          AND znumpp = t_saida_i-znumpp
          AND ( zcvcont = 'PP01' OR zcvcont = 'PP02' OR zcvcont = 'ZP06' ).
      LOOP AT t_proa.
        IF t_saida_i-roteiro IS INITIAL.
          t_saida_i-roteiro = t_proa-zcentr.
        ELSE.
          CONCATENATE t_saida_i-roteiro t_proa-zcentr INTO t_saida_i-roteiro SEPARATED BY space.
        ENDIF.
      ENDLOOP.
*RDM322490 - GVCA20241122 - Begin
      SELECT SINGLE ernam erdat
        INTO (t_saida_i-ernam, t_saida_i-erdat)
        FROM aufk
        WHERE aufnr = t_saida_i-znumpp.
*RDM322490 - GVCA20241122 - End
      APPEND t_saida_i.
    ENDLOOP.

*RDM 190430 - 21.11.17 - Inicio
  ELSEIF p_value = 'T_SAIDA-OP_LIB_CEN'.

    READ TABLE t_saida INDEX v_tabix.

    LOOP AT t_dad_rel WHERE werks <> t_saida-werks
                        AND matnr  = t_saida-matnr
                        AND codigo = t_saida-codigo
                        AND op_lib IS NOT INITIAL.
      CLEAR t_saida_i.
      IF t_dad_rel-nr_ov IS NOT INITIAL.
        SHIFT t_dad_rel-nr_ov LEFT DELETING LEADING '0'.
        SHIFT t_dad_rel-it_ov LEFT DELETING LEADING '0'.
        CONCATENATE t_dad_rel-nr_ov '/' t_dad_rel-it_ov INTO t_saida_i-ov_it.
      ENDIF.
      t_saida_i-werks   = t_dad_rel-werks.
      t_saida_i-znumpp  = t_dad_rel-or_pro.
      t_saida_i-znumoa  = t_dad_rel-op.
      t_saida_i-peso    = t_dad_rel-op_lib.
      t_saida_i-peca    = t_dad_rel-op_peca.
      t_saida_i-wrk02   = t_dad_rel-werks2.

      CLEAR: t_proa[], t_proa, t_saida_i-roteiro.
      SELECT znumoa znumpp zoperpp zcentr
        INTO TABLE t_proa
        FROM zrnppt_proa
        WHERE znumoa = t_saida_i-znumoa
          AND znumpp = t_saida_i-znumpp
          AND ( zcvcont = 'PP01' OR zcvcont = 'PP02' OR zcvcont = 'ZP06' ).
      LOOP AT t_proa.
        IF t_saida_i-roteiro IS INITIAL.
          t_saida_i-roteiro = t_proa-zcentr.
        ELSE.
          CONCATENATE t_saida_i-roteiro t_proa-zcentr INTO t_saida_i-roteiro SEPARATED BY space.
        ENDIF.
      ENDLOOP.
*RDM322490 - GVCA20241122 - Begin
      SELECT SINGLE ernam erdat
        INTO (t_saida_i-ernam, t_saida_i-erdat)
        FROM aufk
        WHERE aufnr = t_saida_i-znumpp.
*RDM322490 - GVCA20241122 - End
      APPEND t_saida_i.
    ENDLOOP.

  ELSEIF p_value = 'T_SAIDA-OP_ABER_CEN'.

    READ TABLE t_saida INDEX v_tabix.

    LOOP AT t_dad_rel WHERE werks <> t_saida-werks
                        AND matnr  = t_saida-matnr
                        AND codigo = t_saida-codigo
                        AND op_aber IS NOT INITIAL.
      CLEAR t_saida_i.
      IF t_dad_rel-nr_ov IS NOT INITIAL.
        SHIFT t_dad_rel-nr_ov LEFT DELETING LEADING '0'.
        SHIFT t_dad_rel-it_ov LEFT DELETING LEADING '0'.
        CONCATENATE t_dad_rel-nr_ov '/' t_dad_rel-it_ov INTO t_saida_i-ov_it.
      ENDIF.
      t_saida_i-werks   = t_dad_rel-werks.
      t_saida_i-znumpp  = t_dad_rel-or_pro.
      t_saida_i-znumoa  = t_dad_rel-op.
      t_saida_i-peso    = t_dad_rel-op_aber.
      t_saida_i-peca    = t_dad_rel-op_peca.
      t_saida_i-wrk02   = t_dad_rel-werks2.
      CLEAR: t_proa[], t_proa, t_saida_i-roteiro.
      SELECT znumoa znumpp zoperpp zcentr
        INTO TABLE t_proa
        FROM zrnppt_proa
        WHERE znumoa = t_saida_i-znumoa
          AND znumpp = t_saida_i-znumpp
          AND ( zcvcont = 'PP01' OR zcvcont = 'PP02' OR zcvcont = 'ZP06' ).
      LOOP AT t_proa.
        IF t_saida_i-roteiro IS INITIAL.
          t_saida_i-roteiro = t_proa-zcentr.
        ELSE.
          CONCATENATE t_saida_i-roteiro t_proa-zcentr INTO t_saida_i-roteiro SEPARATED BY space.
        ENDIF.
      ENDLOOP.
*RDM322490 - GVCA20241122 - Begin
      SELECT SINGLE ernam erdat
        INTO (t_saida_i-ernam, t_saida_i-erdat)
        FROM aufk
        WHERE aufnr = t_saida_i-znumpp.
*RDM322490 - GVCA20241122 - End
      APPEND t_saida_i.
    ENDLOOP.
*RDM 190430 - 21.11.17 - Fim

  ELSEIF p_value = 'T_SAIDA-PRODTOT'.

    READ TABLE t_saida INDEX v_tabix.

    READ TABLE t_marc WITH KEY matnr = t_saida-matnr
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Início
                               werks = t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 01/08/2014 - Fim
    IF sy-subrc IS INITIAL.
      IF t_marc-strgr <> '25'
* Inclusão - Rodrigo Soeiro - 12/2011 - Inicio
     AND t_marc-strgr <> '20' .
* Fim da Inclusão

        LOOP AT t_pmvm WHERE zmatnr = t_saida-matnr.
          CLEAR t_saida_i.
          IF t_pmvm-zqtreal > 0.
            t_saida_i-peso = t_pmvm-zqtreal.
          ELSE.
            t_saida_i-peso = t_pmvm-zqtreg.
          ENDIF.

          t_saida_i-werks       = t_pmvm-werks.
          t_saida_i-matnr_charg = t_pmvm-zmatnr.
          t_saida_i-charg       = t_pmvm-zcharg.
          t_saida_i-znumpp      = t_pmvm-znumpp.
          t_saida_i-znumoa      = t_pmvm-znumoa.
          t_saida_i-peca        = t_pmvm-zqtpeca * 1000.
          t_saida_i-dt_prod     = t_pmvm-zdatreg.
          APPEND t_saida_i.
        ENDLOOP.

      ELSE.

        LOOP AT t_pmvm WHERE zmatnr      = t_saida-matnr
                         AND zmatcli     = t_saida-codigo
                         AND zcodcliente = t_saida-cod_cli.
          CLEAR t_saida_i.
          IF t_pmvm-zqtreal > 0.
            t_saida_i-peso = t_pmvm-zqtreal.
          ELSE.
            t_saida_i-peso = t_pmvm-zqtreg.
          ENDIF.

          t_saida_i-werks       = t_pmvm-werks.
          t_saida_i-matnr_charg = t_pmvm-zmatnr.
          t_saida_i-charg       = t_pmvm-zcharg.
          t_saida_i-znumpp      = t_pmvm-znumpp.
          t_saida_i-znumoa      = t_pmvm-znumoa.
          t_saida_i-peca        = t_pmvm-zqtpeca * 1000.
          t_saida_i-dt_prod     = t_pmvm-zdatreg.
          APPEND t_saida_i.
        ENDLOOP.

      ENDIF.

    ENDIF.

  ELSEIF p_value = 'T_SAIDA-MATERIA_PRIMA'.

    LOOP AT t_stpo WHERE matnr = t_saida-matnr.
      t_saida_i-matnr         = t_saida-matnr.
      t_saida_i-materia_prima = t_stpo-idnrk.
      APPEND t_saida_i.
    ENDLOOP.

  ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 23/11/2015 - Início
  CASE p_value.
    WHEN 'T_SAIDA-REQUISICOES'.
      LOOP AT t_eban INTO wa_eban WHERE matnr = t_saida-matnr
                                          AND werks = t_saida-wrk02.
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Início
*        CLEAR v_kunnr.
*        SELECT kunnr UP TO 1 ROWS
*          INTO v_kunnr
*          FROM vbak
*          WHERE vbeln = wa_eban-zzvbeln.
*        ENDSELECT.
*        IF v_kunnr IS INITIAL.
*
*          CLEAR v_pni.
*          SELECT SINGLE partnumber
*            FROM zqmt_partn_mrp
*            INTO v_pni
*           WHERE vbeln = wa_eban-zzvbeln.
*
*          SELECT SINGLE kunnr
*            FROM zqmt_partnumber
*            INTO v_kunnr
*           WHERE partnumber = v_pni AND
*                 atnam      = 'SD_MATERIAL_CLIENTE' AND
*               ( status     = 'LIB' OR status = 'LIR' ).
*        ENDIF.
*
*        CLEAR v_kdmat.
*        SELECT kdmat UP TO 1 ROWS
*          INTO v_kdmat
*          FROM vbap
*          WHERE vbeln = wa_eban-zzvbeln
*            AND posnr = wa_eban-zzposnr.
*        ENDSELECT.
        PERFORM f_verifica_estrategia USING t_saida-matnr
                                            t_saida-wrk02
                                      CHANGING vl_strgr.
        IF vl_strgr EQ '25'.

          CLEAR v_kdmat.
          SELECT SINGLE kdmat
            FROM vbap
            INTO v_kdmat
            WHERE vbeln = wa_eban-zzvbeln AND
                  posnr = wa_eban-zzposnr.

          CLEAR: v_kunnr.
          SELECT SINGLE kunnr
            INTO v_kunnr
            FROM vbak
           WHERE vbeln = wa_eban-zzvbeln.

          CLEAR t_zrnppt_prev.
          READ TABLE t_zrnppt_prev WITH KEY werks  = t_saida-wrk02
                                            matnr  = t_saida-matnr
                                            codigo = v_kdmat
                                            kunnr  = v_kunnr.
          IF sy-subrc EQ 0.
            CLEAR: v_kunnr, v_kdmat.
          ENDIF.

        ELSE.

          CLEAR t_zrnppt_prev.
          READ TABLE t_zrnppt_prev WITH KEY werks = t_saida-wrk02
                                            matnr = t_saida-matnr.
          IF sy-subrc EQ 0.
            v_kunnr = t_zrnppt_prev-kunnr.
            v_kdmat = t_zrnppt_prev-codigo.
          ENDIF.

        ENDIF.

***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Fim
        IF t_saida-cod_cli <> v_kunnr.
          CONTINUE.
        ENDIF.

        IF t_saida-codigo <> v_kdmat.
          CONTINUE.
        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Início
        IF wa_eban-meins = 'PC' OR wa_eban-meins = 'ST'.
          IF ( t_saida-peso_peca > 0 ).
            wa_eban-menge = wa_eban-menge * ( t_saida-peso_peca / 1000 ).
          ELSE.
            CLEAR wa_eban-menge.
          ENDIF.
        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Fim
        t_saida_i-werks      = wa_eban-werks.      "centro
        t_saida_i-requisicao = wa_eban-banfn.      "requisição
        CONCATENATE  wa_eban-zzvbeln '/' wa_eban-zzposnr INTO t_saida_i-ov_it.   "ov/it
        t_saida_i-peso       = wa_eban-menge. "peso

*RDM 160093
        CLEAR t_saida_i-num_pl_corte.
        SELECT SINGLE num_pl_corte
          FROM zrnppt_pl_itens
          INTO t_saida_i-num_pl_corte
          WHERE werks   = t_saida_i-werks
            AND num_req = t_saida_i-requisicao.
*RDM 160093

        APPEND t_saida_i.
        CLEAR t_saida_i.
      ENDLOOP.

    WHEN 'T_SAIDA-PEDIDOS'.

      LOOP AT t_ekko INTO wa_ekko WHERE matnr = t_saida-matnr
                                    AND werks = t_saida-werks.

*INC 3258470
*        CLEAR: wa_eban.
*        READ TABLE t_eban_aux INTO wa_eban WITH KEY ebeln = wa_ekko-ebeln
*                                                    ebelp = wa_ekko-ebelp
*                                                    BINARY SEARCH.
*INC 3258470

***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Início
*        CLEAR v_kunnr.
*        SELECT kunnr UP TO 1 ROWS
*          INTO v_kunnr
*          FROM vbak
*          WHERE vbeln = wa_eban-zzvbeln.
*        ENDSELECT.
*
*        IF v_kunnr IS INITIAL.
*          CLEAR v_pni.
*          SELECT SINGLE partnumber
*            FROM zqmt_partn_mrp
*            INTO v_pni
*           WHERE vbeln = wa_eban-zzvbeln.
*
*          SELECT SINGLE kunnr
*            FROM zqmt_partnumber
*            INTO v_kunnr
*           WHERE partnumber = v_pni AND
*                 atnam      = 'SD_MATERIAL_CLIENTE' AND
*               ( status     = 'LIB' OR status = 'LIR' ).
*        ENDIF.
*
*        CLEAR v_kdmat.
*        SELECT kdmat UP TO 1 ROWS
*          INTO v_kdmat
*          FROM vbap
*          WHERE vbeln = wa_eban-zzvbeln
*            AND posnr = wa_eban-zzposnr.
*        ENDSELECT.

*INC 3258470
*        PERFORM f_verifica_estrategia USING t_saida-matnr
*                                            t_saida-wrk02
*                                      CHANGING vl_strgr.
*        IF vl_strgr EQ '25'.
*
*          CLEAR v_kdmat.
*          SELECT SINGLE kdmat
*            FROM vbap
*            INTO v_kdmat
*            WHERE vbeln = wa_eban-zzvbeln AND
*                  posnr = wa_eban-zzposnr.
*
*          CLEAR: v_kunnr.
*          SELECT SINGLE kunnr
*            INTO v_kunnr
*            FROM vbak
*           WHERE vbeln = wa_eban-zzvbeln.
*
*          CLEAR t_zrnppt_prev.
*          READ TABLE t_zrnppt_prev WITH KEY werks  = t_saida-wrk02
*                                            matnr  = t_saida-matnr
*                                            codigo = v_kdmat
*                                            kunnr  = v_kunnr.
*          IF sy-subrc EQ 0.
*            CLEAR: v_kunnr, v_kdmat.
*          ENDIF.
*
*        ELSE.
*INC 3258470
        CLEAR t_zrnppt_prev.
        READ TABLE t_zrnppt_prev WITH KEY werks = t_saida-wrk02
                                          matnr = t_saida-matnr.
        IF sy-subrc EQ 0.
          v_kunnr = t_zrnppt_prev-kunnr.
          v_kdmat = t_zrnppt_prev-codigo.
        ENDIF.

*INC 3258470
*        ENDIF.
*INC 3258470

***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Fim
        IF t_saida-cod_cli <> v_kunnr.
          CONTINUE.
        ENDIF.

        IF t_saida-codigo <> v_kdmat.
          CONTINUE.
        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Início
        IF wa_ekko-meins = 'PC' OR wa_ekko-meins = 'ST'.
          IF ( t_saida-peso_peca > 0 ).
            wa_ekko-menge = wa_ekko-menge * ( t_saida-peso_peca / 1000 ).
          ELSE.
            CLEAR wa_ekko-menge.
          ENDIF.
        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Fim
        CLEAR lc_entrada.
        LOOP AT t_ekbe WHERE ebeln = wa_ekko-ebeln AND
                             ebelp = wa_ekko-ebelp.
***Inserção - Hermiro Carvalho - INC 2410544 - 25/08/2017 - Início
          CLEAR: wa_mseg_log, v_answer.
          PERFORM verifica_tipo_movimento USING t_ekbe-matnr
                                                t_ekbe-charg
                                                t_ekbe-belnr
                                                t_ekbe-buzei
                                       CHANGING v_answer.
          IF v_answer IS NOT INITIAL.
            CONTINUE.
          ENDIF.
***Inserção - Hermiro Carvalho - INC 2410544 - 25/08/2017 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Início
          IF wa_ekko-meins = 'PC' OR wa_ekko-meins = 'ST'.
            IF ( t_saida-peso_peca > 0 ).
              t_ekbe-menge = t_ekbe-menge * ( t_saida-peso_peca / 1000 ).
            ELSE.
              CLEAR t_ekbe-menge.
            ENDIF.
          ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Fim
          lc_entrada = t_ekbe-menge + lc_entrada.
        ENDLOOP.

        t_saida_i-werks      = wa_ekko-werks. "Centro
        t_saida_i-pedido     = wa_ekko-ebeln. "Pedido
        CONCATENATE  wa_eban-zzvbeln '/' wa_eban-zzposnr INTO t_saida_i-ov_it. "OV/IT
        t_saida_i-peso       = wa_ekko-menge. "Peso
        t_saida_i-peso_ret   = lc_entrada.  "Peso Retornado
***Inserção - Hermiro Carvalho - RDM 158724 - 26/02/2016 - Início
        t_saida_i-saldo_pedido = t_saida_i-peso - t_saida_i-peso_ret.
***Inserção - Hermiro Carvalho - RDM 158724 -  26/02/2016 - Fim

*RDM 160093
        CLEAR t_saida_i-num_pl_corte.
        SELECT SINGLE num_pl_corte
          FROM zrnppt_pl_itens
          INTO t_saida_i-num_pl_corte
          WHERE werks = t_saida_i-werks
            AND ebeln = t_saida_i-pedido.
*RDM 160093
***Inserção - Hermiro Carvalho - RDM 184185 - 18/09/2017 - Início
        IF t_saida_i-num_pl_corte IS NOT INITIAL.
          CLEAR: wa_pl_corte, wa_lfa1.
          SELECT SINGLE *
            INTO wa_pl_corte
            FROM zrnppt_pl_corte
           WHERE num_pl_corte = t_saida_i-num_pl_corte.
          IF sy-subrc IS INITIAL.
            SELECT SINGLE *
              INTO wa_lfa1
              FROM lfa1
             WHERE lifnr =  wa_pl_corte-lifnr.
            IF sy-subrc IS INITIAL.
              t_saida_i-sortl = wa_lfa1-sortl.
            ENDIF.
          ENDIF.
        ENDIF.
***Inserção - Hermiro Carvalho - RDM 184185 - 18/09/2017 - Fim
*RDM322490 - GVCA20241122 - Begin
        SELECT SINGLE ernam erdat
          INTO (t_saida_i-ernam, t_saida_i-erdat)
          FROM aufk
          WHERE aufnr = t_saida_i-znumpp.
*RDM322490 - GVCA20241122 - End
        APPEND t_saida_i.
        CLEAR t_saida_i.
      ENDLOOP.

*PP - Criar coluna POs Outros Centros na GD - RDM 244981
    WHEN 'T_SAIDA-PEDIDOS_CEN'.

      LOOP AT t_ekko INTO wa_ekko WHERE matnr EQ t_saida-matnr
                                    AND werks NE t_saida-werks.

*INC 3258470
*        CLEAR: wa_eban.
*        READ TABLE t_eban_aux INTO wa_eban WITH KEY ebeln = wa_ekko-ebeln
*                                                    ebelp = wa_ekko-ebelp
*                                                    BINARY SEARCH.
*
*        PERFORM f_verifica_estrategia USING t_saida-matnr
*                                            t_saida-werks
*                                      CHANGING vl_strgr.
*        IF vl_strgr EQ '25'.
*
*          CLEAR v_kdmat.
*          SELECT SINGLE kdmat
*            FROM vbap
*            INTO v_kdmat
*            WHERE vbeln = wa_eban-zzvbeln AND
*                  posnr = wa_eban-zzposnr.
*
*          CLEAR: v_kunnr.
*          SELECT SINGLE kunnr
*            INTO v_kunnr
*            FROM vbak
*           WHERE vbeln = wa_eban-zzvbeln.
*
*          CLEAR t_zrnppt_prev.
*          READ TABLE t_zrnppt_prev WITH KEY werks  = t_saida-werks
*                                            matnr  = t_saida-matnr
*                                            codigo = v_kdmat
*                                            kunnr  = v_kunnr.
*          IF sy-subrc EQ 0.
*            CLEAR: v_kunnr, v_kdmat.
*          ENDIF.
*
*        ELSE.
*INC 3258470
        CLEAR t_zrnppt_prev.
        READ TABLE t_zrnppt_prev WITH KEY werks = t_saida-werks
                                          matnr = t_saida-matnr.
        IF sy-subrc EQ 0.
          v_kunnr = t_zrnppt_prev-kunnr.
          v_kdmat = t_zrnppt_prev-codigo.
        ENDIF.
*INC 3258470
*        ENDIF.
*INC 3258470

        IF t_saida-cod_cli <> v_kunnr.
          CONTINUE.
        ENDIF.

        IF t_saida-codigo <> v_kdmat.
          CONTINUE.
        ENDIF.

        IF wa_ekko-meins = 'PC' OR wa_ekko-meins = 'ST'.
          IF ( t_saida-peso_peca > 0 ).
            wa_ekko-menge = wa_ekko-menge * ( t_saida-peso_peca / 1000 ).
          ELSE.
            CLEAR wa_ekko-menge.
          ENDIF.
        ENDIF.

        CLEAR lc_entrada.
        LOOP AT t_ekbe WHERE ebeln = wa_ekko-ebeln AND
                             ebelp = wa_ekko-ebelp.

          CLEAR: wa_mseg_log, v_answer.
          PERFORM verifica_tipo_movimento USING t_ekbe-matnr
                                                t_ekbe-charg
                                                t_ekbe-belnr
                                                t_ekbe-buzei
                                       CHANGING v_answer.
          IF v_answer IS NOT INITIAL.
            CONTINUE.
          ENDIF.

          IF wa_ekko-meins = 'PC' OR wa_ekko-meins = 'ST'.
            IF ( t_saida-peso_peca > 0 ).
              t_ekbe-menge = t_ekbe-menge * ( t_saida-peso_peca / 1000 ).
            ELSE.
              CLEAR t_ekbe-menge.
            ENDIF.
          ENDIF.

          lc_entrada = t_ekbe-menge + lc_entrada.
        ENDLOOP.

        t_saida_i-werks      = wa_ekko-werks. "Centro
        t_saida_i-pedido     = wa_ekko-ebeln. "Pedido
        CONCATENATE  wa_eban-zzvbeln '/' wa_eban-zzposnr INTO t_saida_i-ov_it. "OV/IT
        t_saida_i-peso       = wa_ekko-menge. "Peso
        t_saida_i-peso_ret   = lc_entrada.  "Peso Retornado
        t_saida_i-saldo_pedido = t_saida_i-peso - t_saida_i-peso_ret.

        CLEAR t_saida_i-num_pl_corte.
        SELECT SINGLE num_pl_corte
          FROM zrnppt_pl_itens
          INTO t_saida_i-num_pl_corte
          WHERE werks = t_saida_i-werks
            AND ebeln = t_saida_i-pedido.

        IF t_saida_i-num_pl_corte IS NOT INITIAL.
          CLEAR: wa_pl_corte, wa_lfa1.
          SELECT SINGLE *
            INTO wa_pl_corte
            FROM zrnppt_pl_corte
           WHERE num_pl_corte = t_saida_i-num_pl_corte.
          IF sy-subrc IS INITIAL.
            SELECT SINGLE *
              INTO wa_lfa1
              FROM lfa1
             WHERE lifnr =  wa_pl_corte-lifnr.
            IF sy-subrc IS INITIAL.
              t_saida_i-sortl = wa_lfa1-sortl.
            ENDIF.
          ENDIF.
        ENDIF.

*RDM322490 - GVCA20241122 - Begin
        SELECT SINGLE ernam erdat
          INTO (t_saida_i-ernam, t_saida_i-erdat)
          FROM aufk
          WHERE aufnr = t_saida_i-znumpp.
*RDM322490 - GVCA20241122 - End
        APPEND t_saida_i.
        CLEAR t_saida_i.

      ENDLOOP.
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
    WHEN OTHERS.
  ENDCASE.
***Inserção - Hermiro Carvalho - PRJ01313 - 23/11/2015 - Fim
ENDFORM.                    "   AJUSTA_TAB_I

*&---------------------------------------------------------------------*
*&      Form  F_USER_COMMAND
*&---------------------------------------------------------------------*
FORM f_user_command USING ok_code     LIKE sy-ucomm
                          rs_selfield TYPE slis_selfield.

  DATA:  ref_grid      TYPE REF TO cl_gui_alv_grid.

  IF ref_grid IS INITIAL.
    CALL FUNCTION 'GET_GLOBALS_FROM_SLVC_FULLSCR'
      IMPORTING
        e_grid = ref_grid.
  ENDIF.

  IF NOT ref_grid IS INITIAL.
    CALL METHOD ref_grid->check_changed_data.
  ENDIF.


  DATA: lt_fcat     TYPE lvc_t_fcat,
        ls_fcat     LIKE LINE OF lt_fcat,
        lt_det      TYPE lvc_t_detm,
        v_tab_aux   TYPE sy-tabix,
        ls_det      TYPE lvc_s_detm,
        wa_mchb2    LIKE LINE OF t_mchb_alv,
        lt_excl     TYPE slis_t_extab,
        ls_excl     TYPE slis_extab,
        wa_resb     TYPE resb,
        vl_option   TYPE c,
        vl_erro     TYPE char01,
        vl_retorno  TYPE string,
        lv_retorno  TYPE xfeld,
        lv_ref_grid TYPE REF TO cl_gui_alv_grid,
        wa_opt      TYPE ctu_params,
        gt_fieldcat TYPE slis_t_fieldcat_alv WITH HEADER LINE.

  DATA: lv_saldoest     TYPE  labst,
        lv_qtde_reserva TYPE  labst,
        lv_qtde_estoque TYPE  labst,
        lv_soma         TYPE  labst.

  IF op2 = 'X'.
    REFRESH: t_saida_aux.
    MOVE-CORRESPONDING t_saida TO t_saida_aux.
    APPEND t_saida_aux.

    CASE sy-ucomm.
*PP - Otimização de Estoque - RDM 255762
      WHEN 'LISTA_ALT'.
        CALL TRANSACTION 'ZPP_MANUT_LISTAALT'.
*        AUTHORITY-CHECK OBJECT 'ZPP_LT_ACT' ID 'ACTVT' DUMMY.
*        IF sy-subrc EQ 0.
*          CALL TRANSACTION 'ZPP_MANUT_LISTAALT'.
*        ELSE.
*          MESSAGE e000(su) WITH 'Sem autorização para realizar manutenção!'(014).
*        ENDIF.
*PP - Otimização de Estoque - RDM 255762
      WHEN 'GATILHO_JT'.
        CALL TRANSACTION 'ZPP_JIT_CONSIGNADO'.

      WHEN 'SAVE'.
        PERFORM pf_save_zrnppt_prev.

      WHEN 'CANC' OR 'EXIT'.
        LEAVE PROGRAM.
      WHEN 'MMBE' OR 'CS03' OR 'MM03' OR 'CA03'.
        CLEAR v_cont.
        LOOP AT t_saida WHERE mark = 'X'.
          ADD 1 TO v_cont.
        ENDLOOP.

        IF v_cont IS INITIAL.
          MESSAGE 'Selecionar 1 linha' TYPE 'I'.
        ELSEIF v_cont > '1'.
          MESSAGE 'Selecionar apenas 1 linha' TYPE 'I'.
        ELSEIF v_cont = '1'.

          READ TABLE t_saida WITH KEY mark = 'X'.

          SET PARAMETER ID: 'MAT' FIELD t_saida-matnr.
          IF sy-ucomm = 'MMBE'.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*            SET PARAMETER ID: 'WRK' FIELD p_centro-low.
            SET PARAMETER ID: 'WRK' FIELD t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
            CALL TRANSACTION 'MMBE' AND SKIP FIRST SCREEN.
          ELSEIF sy-ucomm = 'CS03'.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*            SET PARAMETER ID: 'WRK' FIELD p_centro-low.
            SET PARAMETER ID: 'WRK' FIELD t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
            SET PARAMETER ID: 'CSV' FIELD '1'.
            CALL TRANSACTION 'CS03' AND SKIP FIRST SCREEN.
          ELSEIF sy-ucomm = 'MM03'.
            CALL TRANSACTION 'MM03' AND SKIP FIRST SCREEN.
          ELSEIF sy-ucomm = 'CA03'.
            SET PARAMETER ID: 'WRK' FIELD t_saida-wrk02.
            CALL TRANSACTION 'CA03' AND SKIP FIRST SCREEN.
          ENDIF.

        ENDIF.

***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
      WHEN 'ZMAPA'.
        CLEAR v_cont.
        LOOP AT t_saida WHERE mark = 'X'.
          ADD 1 TO v_cont.
        ENDLOOP.

        IF v_cont IS INITIAL.
          MESSAGE 'Selecionar 1 linha' TYPE 'I'.
        ELSEIF v_cont > '1'.
          MESSAGE 'Selecionar apenas 1 linha' TYPE 'I'.
        ELSEIF v_cont = '1'.
          READ TABLE t_saida WITH KEY mark = 'X'.
          READ TABLE t_zrnppt_prev WITH KEY bismt  = t_saida-cliente
                                            matnr  = t_saida-matnr
                                            codigo = t_saida-codigo.

          SUBMIT zsuppr_mapa WITH s_werks EQ t_saida-werks "Centro.
                             WITH s_kunnr EQ t_saida-cliente "Cliente
                             WITH s_matnr EQ t_saida-matnr "Material
                             WITH p_mes       EQ t_zrnppt_prev-mes"Mes
                             WITH p_ano       EQ t_zrnppt_prev-ano USING SELECTION-SCREEN 1000"Mes
*                             WITH p_todos     EQ 'X' USING SELECTION-SCREEN 1000
                                   AND RETURN.
        ENDIF.
      WHEN 'CORTE'.

        PERFORM plano_de_corte.

***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Início
      WHEN 'COMPO'.
        PERFORM composicoes.
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Fim
***Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Fim

* INICIO: RGC8096 06mar2018 RDM
      WHEN 'SKU'.
        READ TABLE t_saida WITH KEY mark = c_x.
        IF sy-subrc <> 0.
          MESSAGE text-015 TYPE c_e.
        ENDIF.
        PERFORM sku USING rs_selfield-value.
* INICIO: RGC8096 06mar2018 RDM
***Inserção - Hermiro Carvalho - RDM 196192 - 20/06/2018 - Início
      WHEN 'ATUA'."Atualiza as informações da tela sem voltar a tela de seleção.
        PERFORM atualiza_alv.
        CLEAR lv_ref_grid.
        PERFORM progress_indicator USING '99' 'Gerando Relatório...'.
        IF lv_ref_grid IS INITIAL.
          CALL FUNCTION 'GET_GLOBALS_FROM_SLVC_FULLSCR'
            IMPORTING
              e_grid = lv_ref_grid.
        ENDIF.
        IF lv_ref_grid IS NOT INITIAL.
          CALL METHOD lv_ref_grid->check_changed_data.
          rs_selfield-refresh = 'X'.
        ENDIF.
***Inserção - Hermiro Carvalho - RDM 196192 - 20/06/2018 - Fim
      WHEN OTHERS.

        IF sy-ucomm IS INITIAL.
          READ TABLE t_saida INDEX rs_selfield-tabindex.
          IF rs_selfield-sel_tab_field = 'T_SAIDA-MATNR'.
*             rs_selfield-sel_tab_field = 'T_SAIDA-PLANEJADAS'.
*             rs_selfield-sel_tab_field = 'T_SAIDA-REQUISICOES' OR
*             rs_selfield-sel_tab_field = 'T_SAIDA-PEDIDOS'.
            SET PARAMETER ID: 'MAT' FIELD rs_selfield-value.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
*            SET PARAMETER ID: 'WRK' FIELD p_centro-low.
            SET PARAMETER ID: 'WRK' FIELD t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim

            CALL TRANSACTION 'MD04' AND SKIP FIRST SCREEN.

          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-EXPEDICAO'  OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-EXP_INTER'  OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-MAT_CQ'     OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-EXPEDI_CEN' OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-FATURADO'   OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-FATURADO1'   OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-FATURADO_CEN' OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-FATURADO1_CEN' OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-OP_LIB'     OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-OP_ABER'    OR
*RDM 190430 - 21.11.17 - Inicio
                 rs_selfield-sel_tab_field = 'T_SAIDA-OP_LIB_CEN'  OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-OP_ABER_CEN' OR
*RDM 190430 - 21.11.17 - Fim
                 rs_selfield-sel_tab_field = 'T_SAIDA-PRODTOT'    OR
***Inserção - Hermiro Carvalho - PRJ01313 - 23/11/2015 - Início
                 rs_selfield-sel_tab_field = 'T_SAIDA-REQUISICOES' OR
                 rs_selfield-sel_tab_field = 'T_SAIDA-PEDIDOS' OR
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
                 rs_selfield-sel_tab_field = 'T_SAIDA-PEDIDOS_CEN' OR
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
***Inserção - Hermiro Carvalho - PRJ01313 - 23/11/2015 - Fim
                ( rs_selfield-sel_tab_field = 'T_SAIDA-MATERIA_PRIMA' AND rs_selfield-value = '*' ).  "Só entra para matéria prima se tiver mais de uma matéria prima

            FREE t_saida_i[].
            v_tabix = rs_selfield-tabindex.
            PERFORM ajusta_tab_i USING rs_selfield-sel_tab_field.
            IF t_saida_i[] IS NOT INITIAL.
              PERFORM f_imprime_saida_i USING rs_selfield-sel_tab_field.
            ELSEIF rs_selfield-value IS NOT INITIAL.
              MESSAGE i897(sd) WITH text-009.
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 140328 - 22/06/2015 - Início
*          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-NECESSIDADE_MP'.
*
*            READ TABLE t_saida INDEX rs_selfield-tabindex.
*
*            IF sy-subrc IS INITIAL AND t_saida-materia_mp IS NOT INITIAL.
*              SET PARAMETER ID: 'MAT' FIELD t_saida-materia_mp.
****Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
**            SET PARAMETER ID: 'WRK' FIELD p_centro-low.
*              SET PARAMETER ID: 'WRK' FIELD t_saida-werks.
****Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim
*              CALL TRANSACTION 'MMBE' AND SKIP FIRST SCREEN.
*            ELSEIF t_saida-materia_mp IS INITIAL.
*              MESSAGE 'Matéria prima não encontrada' TYPE 'I'.
*            ELSE.
*              MESSAGE 'Não encontrado' TYPE 'I'.
*            ENDIF.
*          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_MP'.
*            READ TABLE t_saida INDEX rs_selfield-tabindex.
*            SET PARAMETER ID: 'MAT' FIELD t_saida-materia_mp.
**            SET PARAMETER ID: 'WRK' FIELD t_saida-werks.
*            CALL TRANSACTION 'MMBE' AND SKIP FIRST SCREEN.

***Inserção - LG - RDM 334177 - 15/04/2026 - Início - Drilldown ZZOPSEMMOV
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ZZOPSEMMOV'.

            IF p_corte IS INITIAL.
              MESSAGE 'Drilldown disponível apenas na opção de cobertura superior a 3 meses' TYPE 'I'.
              RETURN.
            ENDIF.

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

***Inserção - RDM 334150 - Início - Enriquecimento campos novos popup OP
            IF gt_ef_op_det[] IS NOT INITIAL.

              DATA: lt_pmvm    TYPE TABLE OF zrnppt_pmvm,
                    ls_pmvm    TYPE zrnppt_pmvm,
                    lt_papo    TYPE TABLE OF zrnppt_papo,
                    ls_papo    TYPE zrnppt_papo,
                    lt_dpo     TYPE TABLE OF zrnppt_dpo,
                    ls_dpo     TYPE zrnppt_dpo,
                    lt_matint  TYPE TABLE OF zsuppt_matinterm,
                    ls_matint  TYPE zsuppt_matinterm,
                    lt_jest    TYPE TABLE OF jest,
                    ls_jest    TYPE jest,
                    lt_caufv   TYPE TABLE OF caufv,
                    ls_caufv   TYPE caufv,
                    lv_prod_ac TYPE zrnppt_pmvm-zqtreg,
                    lv_conf_ac TYPE zrnppt_papo-zqtprd.

              " Centro - via CAUFV a partir do AUFNR
              SELECT aufnr werks
                INTO CORRESPONDING FIELDS OF TABLE lt_caufv
                FROM caufv
                FOR ALL ENTRIES IN gt_ef_op_det
               WHERE aufnr = gt_ef_op_det-aufnr.

              " Produzido - ZRNPPT_PMVM (ZNUMPP = AUFNR)
              SELECT znumpp zqtreg
                INTO CORRESPONDING FIELDS OF TABLE lt_pmvm
                FROM zrnppt_pmvm
                FOR ALL ENTRIES IN gt_ef_op_det
               WHERE znumpp = gt_ef_op_det-aufnr.

              " Confirmação de Produção - ZRNPPT_PAPO (ZNUMPP = AUFNR)
              SELECT znumpp zqtprd
                INTO CORRESPONDING FIELDS OF TABLE lt_papo
                FROM zrnppt_papo
                FOR ALL ENTRIES IN gt_ef_op_det
               WHERE znumpp = gt_ef_op_det-aufnr.

              " Diário - ZRNPPT_DPO (AUFNR direto - campo NUMDIAR é o Diário)
              SELECT aufnr numdiar
                INTO CORRESPONDING FIELDS OF TABLE lt_dpo
                FROM zrnppt_dpo
                FOR ALL ENTRIES IN gt_ef_op_det
               WHERE aufnr = gt_ef_op_det-aufnr.

              " Status da operação - JEST (via OBJNR da OP)
              SELECT objnr stat inact
                INTO CORRESPONDING FIELDS OF TABLE lt_jest
                FROM jest
                FOR ALL ENTRIES IN gt_ef_op_det
               WHERE objnr = gt_ef_op_det-objnr
                 AND inact = ''.

              " 1ª passada - preenche WERKS, Produzido, Confirmação, Diário, Status
              LOOP AT gt_ef_op_det ASSIGNING FIELD-SYMBOL(<fs_op>).
                "FIELD-SYMBOL para atualização in-place (evita cópia + MODIFY)

                CLEAR: lv_prod_ac, lv_conf_ac.

                " Centro
                READ TABLE lt_caufv INTO ls_caufv WITH KEY aufnr = <fs_op>-aufnr.
                IF sy-subrc = 0.
                  <fs_op>-werks = ls_caufv-werks.
                ENDIF.

                " Planejado (mesmo valor de QTD - exibição explícita)
                <fs_op>-planejado = <fs_op>-qtd.

                " Produzido - soma ZQTREG
                LOOP AT lt_pmvm INTO ls_pmvm WHERE znumpp = <fs_op>-aufnr.
                  lv_prod_ac = lv_prod_ac + ls_pmvm-zqtreg.
                ENDLOOP.
                <fs_op>-produzido = lv_prod_ac.

                " Confirmação de Produção - soma ZQTPRD
                LOOP AT lt_papo INTO ls_papo WHERE znumpp = <fs_op>-aufnr.
                  lv_conf_ac = lv_conf_ac + ls_papo-zqtprd.
                ENDLOOP.
                <fs_op>-confirmacao = lv_conf_ac.

                " Diário - último NUMDIAR encontrado
                READ TABLE lt_dpo INTO ls_dpo WITH KEY aufnr = <fs_op>-aufnr.
                IF sy-subrc = 0.
                  <fs_op>-diario = ls_dpo-numdiar.
                ENDIF.

                " Status - traduz JEST
                READ TABLE lt_jest INTO ls_jest WITH KEY objnr = <fs_op>-objnr
                                                         stat  = 'I0002'.
                IF sy-subrc = 0.
                  <fs_op>-status_op = 'LIB'.
                ELSE.
                  READ TABLE lt_jest INTO ls_jest WITH KEY objnr = <fs_op>-objnr
                                                           stat  = 'I0001'.
                  IF sy-subrc = 0.
                    <fs_op>-status_op = 'ABER'.
                  ENDIF.
                ENDIF.

              ENDLOOP.

              " 2ª passada - Intermediário (depende do WERKS já preenchido)
              SELECT werks matnr
                INTO CORRESPONDING FIELDS OF TABLE lt_matint
                FROM zsuppt_matinterm
                FOR ALL ENTRIES IN gt_ef_op_det
               WHERE matnr = gt_ef_op_det-matnr
                 AND werks = gt_ef_op_det-werks.

              LOOP AT gt_ef_op_det ASSIGNING <fs_op> WHERE werks IS NOT INITIAL.
                READ TABLE lt_matint TRANSPORTING NO FIELDS
                  WITH KEY matnr = <fs_op>-matnr
                           werks = <fs_op>-werks.
                IF sy-subrc = 0.
                  <fs_op>-intermed = 'X'.
                ENDIF.
              ENDLOOP.

            ENDIF.
***Inserção - RDM 334150 - Fim

            IF gt_ef_op_det[] IS INITIAL.
              MESSAGE 'Não há OP sem movimentação para o item selecionado' TYPE 'I'.
              RETURN.
            ENDIF.

            gt_fieldcat-fieldname = 'AUFNR'.     gt_fieldcat-seltext_m = 'Nº PP'.           gt_fieldcat-hotspot = 'X'. APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'MATNR'.     gt_fieldcat-seltext_m = 'Material'.        APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'QTD'.       gt_fieldcat-seltext_m = 'Quantidade'.      APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'ERDAT'.     gt_fieldcat-seltext_m = 'Data Abertura'.   APPEND gt_fieldcat. CLEAR gt_fieldcat.

            gt_fieldcat-fieldname = 'WERKS'.     gt_fieldcat-seltext_m = 'Centro'.          APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'DIARIO'.    gt_fieldcat-seltext_m = 'Diário'.         APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'PLANEJADO'. gt_fieldcat-seltext_m = 'Planejado'.      APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'PRODUZIDO'. gt_fieldcat-seltext_m = 'Produzido'.      APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'INTERMED'.  gt_fieldcat-seltext_m = 'Intermediário'.  APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'CONFIRMACAO'. gt_fieldcat-seltext_m = 'Conf.Prod'.    APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'STATUS_OP'. gt_fieldcat-seltext_m = 'Status'.         APPEND gt_fieldcat. CLEAR gt_fieldcat.


            CLEAR gs_layout.
            gs_layout-no_input          = 'X'.
            gs_layout-zebra             = 'X'.
            gs_layout-colwidth_optimize = 'X'.

            CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
              EXPORTING
                i_callback_program       = sy-repid
                i_callback_pf_status_set = 'F_OP_DET_PF_STATUS'
                i_callback_user_command  = 'F_OP_DET_USER_COMMAND'
                i_grid_title             = 'Detalhe OP Sem Movimentação'
                is_layout                = gs_layout
                it_fieldcat              = gt_fieldcat[]
                i_save                   = 'X'
              TABLES
                t_outtab                 = gt_ef_op_det
              EXCEPTIONS
                program_error            = 1
                OTHERS                   = 2.
            IF sy-subrc <> 0.
              MESSAGE i000(0k) WITH sy-subrc.
            ENDIF.

***Inserção - RDM 334177 - 2026 - Fim - Drilldown ZZOPSEMMOV (enriquecido)

*            gt_fieldcat-fieldname = 'AUFNR'.
*            gt_fieldcat-seltext_m = 'Nº PP'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            gt_fieldcat-fieldname = 'MATNR'.
*            gt_fieldcat-seltext_m = 'Material'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            gt_fieldcat-fieldname = 'QTD'.
*            gt_fieldcat-seltext_m = 'Quantidade'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            gt_fieldcat-fieldname = 'ERDAT'.
*            gt_fieldcat-seltext_m = 'Data Abertura'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            ls_excl-fcode = '&ETA'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '%SC'.  APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '%SC+'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&OUP'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&ODN'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&ILT'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&OL0'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&CRB'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&CRL'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&CRR'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&CRE'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&AC1'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&ALL'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&SAL'. APPEND ls_excl TO lt_excl.
*
*            CALL FUNCTION 'REUSE_ALV_POPUP_TO_SELECT'
*              EXPORTING
*                i_title               = 'Detalhe OP Sem Movimentação'
*                i_allow_no_selection  = 'X'
*                i_zebra               = 'X'
*                i_screen_start_column = 10
*                i_screen_start_line   = 5
*                i_screen_end_column   = 120
*                i_screen_end_line     = 15
*                i_tabname             = 1
*                it_fieldcat           = gt_fieldcat[]
*                it_excluding          = lt_excl
*                i_callback_program    = sy-repid
*              TABLES
*                t_outtab              = gt_ef_op_det
*              EXCEPTIONS
*                program_error         = 1
*                OTHERS                = 2.
*            IF sy-subrc <> 0.
*              MESSAGE i000(0k) WITH sy-subrc.
*            ENDIF.

***Inserção - LG - RDM 334177 - 15/04/2026 - Fim - Drilldown ZZOPSEMMOV

***Inserção - LG - RDM 334177 - 15/04/2026 - Início - Drilldown ZZPEDSEMMOV
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ZZPEDSEMMOV'.

            IF p_corte IS INITIAL.
              MESSAGE 'Drilldown disponível apenas na opção de cobertura superior a 3 meses' TYPE 'I'.
              RETURN.
            ENDIF.

            CLEAR: gt_ef_ped_det[], gt_fieldcat[], lt_excl[], ls_excl.
            READ TABLE t_saida INDEX rs_selfield-tabindex.

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

              " 3. Enriquece campo NFE: EKPO -> EKBE -> J_1BNFLIN (REFKEY) -> J_1BNFDOC
              IF gt_ef_ped_det IS NOT INITIAL.
                DATA: lt_ekbe_nfe  TYPE TABLE OF ekbe,
                      ls_ekbe_nfe  TYPE ekbe,
                      ls_nflin     TYPE j_1bnflin,
                      ls_nfdoc     TYPE j_1bnfdoc,
                      lv_refkey_nf TYPE j_1bnflin-refkey.

                SELECT ebeln ebelp belnr gjahr buzei
                  INTO CORRESPONDING FIELDS OF TABLE lt_ekbe_nfe
                  FROM ekbe
                  FOR ALL ENTRIES IN gt_ef_ped_det
                  WHERE ebeln = gt_ef_ped_det-ebeln
                    AND ebelp = gt_ef_ped_det-ebelp
                    AND vgabe = '1'.

                LOOP AT gt_ef_ped_det ASSIGNING FIELD-SYMBOL(<fs_ped>).

                  READ TABLE lt_ekbe_nfe ASSIGNING FIELD-SYMBOL(<fs_ekbe>)
                    WITH KEY ebeln = <fs_ped>-ebeln
                             ebelp = <fs_ped>-ebelp.
                  CHECK sy-subrc = 0.

                  lv_refkey_nf = |{ <fs_ekbe>-belnr }{ <fs_ekbe>-gjahr }|.

                  SELECT SINGLE *
                    FROM j_1bnflin
                    INTO ls_nflin
                   WHERE refkey = lv_refkey_nf
                     AND refitm = <fs_ekbe>-buzei.
                  CHECK sy-subrc = 0.

                  SELECT SINGLE nfenum
                    FROM j_1bnfdoc
                    INTO ls_nfdoc-nfenum
                   WHERE docnum = ls_nflin-docnum.
                  IF sy-subrc = 0.
                    <fs_ped>-nfenum = ls_nfdoc-nfenum.
                  ENDIF.

                ENDLOOP.
              ENDIF.

            ENDIF.

            IF gt_ef_ped_det[] IS INITIAL.
              MESSAGE 'Não há pedido sem movimentação para o item selecionado' TYPE 'I'.
              RETURN.
            ENDIF.

            gt_fieldcat-fieldname = 'EBELN'.  gt_fieldcat-seltext_m = 'Nº Pedido'.   gt_fieldcat-hotspot = 'X'. APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'EBELP'.  gt_fieldcat-seltext_m = 'Item Pedido'. APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'MATNR'.  gt_fieldcat-seltext_m = 'Material'.    APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'QTD'.    gt_fieldcat-seltext_m = 'Quantidade'.  APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'DATA'.   gt_fieldcat-seltext_m = 'Data Pedido'. APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'BANFN'.  gt_fieldcat-seltext_m = 'ReqC'.        APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'WERKS'.  gt_fieldcat-seltext_m = 'Centro'.      APPEND gt_fieldcat. CLEAR gt_fieldcat.
            gt_fieldcat-fieldname = 'NFENUM'. gt_fieldcat-seltext_m = 'Nfe'.         APPEND gt_fieldcat. CLEAR gt_fieldcat.

            CLEAR gs_layout.
            gs_layout-no_input          = 'X'.
            gs_layout-zebra             = 'X'.
            gs_layout-colwidth_optimize = 'X'.

            CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
              EXPORTING
                i_callback_program       = sy-repid
                i_callback_pf_status_set = 'F_PED_DET_PF_STATUS'
                i_callback_user_command  = 'F_PED_DET_USER_COMMAND'
                i_grid_title             = 'Detalhe Pedido Sem Movimentação'
                is_layout                = gs_layout
                it_fieldcat              = gt_fieldcat[]
                i_save                   = 'X'
              TABLES
                t_outtab                 = gt_ef_ped_det
              EXCEPTIONS
                program_error            = 1
                OTHERS                   = 2.
            IF sy-subrc <> 0.
              MESSAGE i000(0k) WITH sy-subrc.
            ENDIF.

***Inserção - RDM 334177 - 2026 - Fim - Drilldown ZZPEDSEMMOV (enriquecido)

*            gt_fieldcat-fieldname = 'EBELN'.
*            gt_fieldcat-seltext_m = 'Nº Pedido'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            gt_fieldcat-fieldname = 'EBELP'.
*            gt_fieldcat-seltext_m = 'Item Pedido'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            gt_fieldcat-fieldname = 'MATNR'.
*            gt_fieldcat-seltext_m = 'Material'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            gt_fieldcat-fieldname = 'QTD'.
*            gt_fieldcat-seltext_m = 'Quantidade'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            gt_fieldcat-fieldname = 'DATA'.
*            gt_fieldcat-seltext_m = 'Data Pedido'.
*            APPEND gt_fieldcat.
*            CLEAR gt_fieldcat.
*
*            ls_excl-fcode = '&ETA'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '%SC'.  APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '%SC+'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&OUP'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&ODN'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&ILT'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&OL0'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&CRB'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&CRL'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&CRR'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&CRE'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&AC1'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&ALL'. APPEND ls_excl TO lt_excl.
*            ls_excl-fcode = '&SAL'. APPEND ls_excl TO lt_excl.
*
*            CALL FUNCTION 'REUSE_ALV_POPUP_TO_SELECT'
*              EXPORTING
*                i_title               = 'Detalhe Pedido Sem Movimentação'
*                i_allow_no_selection  = 'X'
*                i_zebra               = 'X'
*                i_screen_start_column = 10
*                i_screen_start_line   = 5
*                i_screen_end_column   = 120
*                i_screen_end_line     = 15
*                i_tabname             = 1
*                it_fieldcat           = gt_fieldcat[]
*                it_excluding          = lt_excl
*                i_callback_program    = sy-repid
*              TABLES
*                t_outtab              = gt_ef_ped_det
*              EXCEPTIONS
*                program_error         = 1
*                OTHERS                = 2.
*            IF sy-subrc <> 0.
*              MESSAGE i000(0k) WITH sy-subrc.
*            ENDIF.

***Inserção - LG - RDM 334177 - 15/04/2026 - Fim - Drilldown ZZPEDSEMMOV

          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_LTAN'.
            CLEAR: t_mchb_aux[], lt_fcat[], ls_fcat, t_hist_lt[],
                   lt_excl[], ls_excl, gt_fieldcat[].
***Seleciona dados na tabela zsuppt_hist_lt
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            SELECT werks matnr idnrk
              INTO TABLE t_hist_lt
              FROM zsuppt_hist_lt
             WHERE werks  EQ t_saida-wrk02
               AND matnr  EQ t_saida-matnr.

            IF sy-subrc IS INITIAL."Verifica se encontrou dados - Início
              SELECT matnr werks lgort charg ersda clabs
                INTO TABLE t_mchb_alv
                FROM mchb
                FOR ALL ENTRIES IN t_hist_lt
               WHERE matnr   EQ t_hist_lt-idnrk
                 AND werks   EQ t_hist_lt-werks
*                 AND lgort   NE r_lgort
                 AND clabs > 0.
              IF t_mchb_alv[] IS NOT INITIAL."Caso a tabela de estoque de lotes(MCHB) não esteja vazia - Início
***Inserção - Hermiro Carvalho - RDM 179464 - 29/05/2017 - Início
                LOOP AT t_mchb_alv INTO wa_mchb_alv.
                  v_tabix = sy-tabix.
                  CLEAR: wa_resb.
                  SELECT SINGLE * FROM resb
                  INTO CORRESPONDING FIELDS OF wa_resb
                  WHERE xloek = ''
                  AND matnr   = wa_mchb_alv-matnr
                  AND werks   = wa_mchb_alv-werks
                  AND lgort   = wa_mchb_alv-lgort
                  AND charg   = wa_mchb_alv-charg
                  AND bwart   = '261'.
                  IF sy-subrc = 0.
*                    wa_resb-aufnr
*Verifica o status da PP
                    SELECT SINGLE aufnr idat2 cy_seqnr
                      INTO (caufv-aufnr, caufv-idat2, caufv-cy_seqnr)
                      FROM caufv
                      WHERE aufnr EQ wa_resb-aufnr.
                    IF sy-subrc EQ 0.
                      wa_mchb_alv-aufnr = caufv-aufnr.
                      IF caufv-idat2 = '00000000'.
                        wa_mchb_alv-znumoa_aber  = caufv-cy_seqnr .
                      ELSE.
                        wa_mchb_alv-znumoa_lib  = caufv-cy_seqnr .
                      ENDIF.
                    ENDIF.
*                    IF wa_mchb_alv-lgort = '0096'.
*                      wa_mchb_alv-cinsm = wa_mchb_alv-clabs.
*                    ENDIF.
*                    MODIFY t_mchb_alv FROM wa_mchb_alv INDEX v_tabix.
                  ENDIF.
                  IF wa_mchb_alv-lgort = '0096'.
                    wa_mchb_alv-cinsm = wa_mchb_alv-clabs.
                  ENDIF.
                  MODIFY t_mchb_alv FROM wa_mchb_alv INDEX v_tabix.
                ENDLOOP.
***Inserção - Hermiro Carvalho - RDM 179464 - 29/05/2017 - Fim
                gt_fieldcat-fieldname = 'MATNR'.
                gt_fieldcat-seltext_m = 'Material'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.

                gt_fieldcat-fieldname = 'WERKS'.
                gt_fieldcat-seltext_m = 'Centro'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.

                gt_fieldcat-fieldname = 'LGORT'.
                gt_fieldcat-seltext_m = 'Depósito'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.

                gt_fieldcat-fieldname = 'CHARG'.
                gt_fieldcat-seltext_m = 'Lote'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.

                gt_fieldcat-fieldname = 'ERSDA'.
                gt_fieldcat-seltext_m = 'Data de Criação'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.

                gt_fieldcat-fieldname = 'CLABS'.
                gt_fieldcat-seltext_m = 'Peso'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.

***Inserção - Hermiro Carvalho - RDM 179464 - 29/05/2017 - Início
                gt_fieldcat-fieldname = 'AUFNR'.
                gt_fieldcat-seltext_m = 'PP'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.

                gt_fieldcat-fieldname = 'ZNUMOA_LIB'.
                gt_fieldcat-seltext_m = 'OP. Lib.'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.


                gt_fieldcat-fieldname = 'ZNUMOA_ABER'.
                gt_fieldcat-seltext_m = 'OP. Aberta'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.

                gt_fieldcat-fieldname = 'CINSM'.
                gt_fieldcat-seltext_m = 'Estoque CQ'.
                APPEND gt_fieldcat.
                CLEAR: gt_fieldcat.
***Inserção - Hermiro Carvalho - RDM 179464 - 29/05/2017 - Fim

                ls_excl-fcode = '&ETA'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '%SC'.  APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '%SC+'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&OUP'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&ODN'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&ILT'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&OL0'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&CRB'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&CRL'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&CRR'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&CRE'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&AC1'. APPEND ls_excl TO lt_excl. " cancel
                ls_excl-fcode = '&ALL'. APPEND ls_excl TO lt_excl.
                ls_excl-fcode = '&SAL'. APPEND ls_excl TO lt_excl.

                CALL FUNCTION 'REUSE_ALV_POPUP_TO_SELECT'
                  EXPORTING
                    i_title               = 'Relatório Detalhado'
                    i_allow_no_selection  = 'X'
                    i_zebra               = 'X'
                    i_screen_start_column = 5
                    i_screen_start_line   = 5
                    i_screen_end_column   = 120
                    i_screen_end_line     = 15
                    i_tabname             = 1
                    it_fieldcat           = gt_fieldcat[]
                    it_excluding          = lt_excl
                    i_callback_program    = sy-repid
                  TABLES
                    t_outtab              = t_mchb_alv
                  EXCEPTIONS
                    program_error         = 1
                    OTHERS                = 2.
                IF sy-subrc <> 0.
                  MESSAGE i000(0k) WITH sy-subrc.
                ENDIF.
              ELSE."Não encontrou registros
                MESSAGE 'Não há histórico para este componente' TYPE 'S' DISPLAY LIKE 'E'.
                RETURN.
              ENDIF."Caso a tabela de estoque de lotes(MCHB) não esteja vazia - Fim
            ELSE."Não se encontrou dados
              MESSAGE 'Não há histórico para este componente' TYPE 'S' DISPLAY LIKE 'E'.
              RETURN.
            ENDIF."Verifica se encontrou dados - Fim

          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_LTAL'.

            DATA: lv_repid TYPE sy-repid,
                  selfield TYPE slis_selfield.

            CLEAR: t_mchb_aux[], lt_fcat[], ls_fcat, t_alternativa[],
                   lt_excl[], ls_excl, gt_fieldcat[].
***Seleciona dados na tabela zsuppt_lista_alt
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            SELECT werks matnr charg matnr_pa obs clabs dt_lib_qa rsnum
              INTO CORRESPONDING FIELDS OF TABLE t_alternativa
              FROM zsuppt_lista_alt
              WHERE werks      EQ t_saida-wrk02
                AND matnr_pa   EQ t_saida-matnr
*PP - Otimização de Estoque - RDM 255762
                AND desativada EQ ''.
*PP - Otimização de Estoque - RDM 255762

            IF sy-subrc IS INITIAL."Verifica se encontrou dados - Início
              SELECT matnr werks lgort charg ersda clabs
                INTO TABLE t_mchb_alv
                FROM mchb
                FOR ALL ENTRIES IN t_alternativa
               WHERE matnr   EQ t_alternativa-matnr
                 AND charg   EQ t_alternativa-charg
                 AND lgort   NE r_lgort
                 AND clabs > 0.

              LOOP AT t_mchb_alv INTO wa_mchb_alv.
                v_tabix = sy-tabix.
                PERFORM f_check_disponibilidade_lote USING  wa_mchb_alv-matnr
                                                            wa_mchb_alv-werks
                                                            wa_mchb_alv-lgort
                                                            wa_mchb_alv-charg
                                                  CHANGING  wa_mchb_alv-clabs.
                IF wa_mchb_alv-clabs LE 0.
                  DELETE t_mchb_alv INDEX v_tabix.
                ENDIF.
              ENDLOOP.

              IF t_mchb_alv[] IS NOT INITIAL."Caso a tabela de estoque de lotes(MCHB) não esteja vazia - Início

                LOOP AT t_alternativa INTO wa_alternativa.

                  v_tab_aux = sy-tabix.
                  READ TABLE t_mchb_alv INTO wa_mchb_alv WITH KEY matnr = wa_alternativa-matnr
                                                                  charg = wa_alternativa-charg.

                  IF sy-subrc NE 0 OR ( sy-subrc = 0 AND wa_mchb_alv-clabs = 0 ).
                    DELETE t_alternativa INDEX v_tab_aux.
                  ENDIF.

                ENDLOOP.

*PP - Melhorias MP Alternativa - RDM 271191
                DATA: lt_pajt_log TYPE STANDARD TABLE OF zppt_pajt_log,
                      ls_pajt_log TYPE zppt_pajt_log.

                SELECT werks matnr charg werks_apl matnr_apl
                       udate utime clabs
                INTO CORRESPONDING FIELDS OF TABLE lt_pajt_log
                FROM zppt_pajt_log
                  FOR ALL ENTRIES IN t_mchb_alv
                WHERE werks EQ t_mchb_alv-werks
                  AND matnr EQ t_mchb_alv-matnr
                  AND charg EQ t_mchb_alv-charg
                  AND werks_apl EQ t_saida-wrk02
                  AND matnr_apl EQ t_saida-matnr
                  AND situacao EQ 'APROVADO'.

                IF sy-subrc IS NOT INITIAL. "Se o registro não for encontrado na tabela zppt_pajt_log, Leia ZPPT_OEST_LOG e inclui na tabela it_pajt_log. Flávio Marqueviski

                  SELECT werks matnr charg werks_apl matnr_apl
                         udate utime clabs
                  INTO CORRESPONDING FIELDS OF TABLE lt_pajt_log
                  FROM zppt_oest_log
                    FOR ALL ENTRIES IN t_mchb_alv
                  WHERE werks EQ t_mchb_alv-werks
                    AND matnr EQ t_mchb_alv-matnr
                    AND charg EQ t_mchb_alv-charg
                    AND werks_apl EQ t_saida-wrk02
                    AND matnr_apl EQ t_saida-matnr
                    AND situacao EQ 'APROVADO'.

                  CLEAR: t_mseg, t_mseg_sum.
                  SELECT m~werks m~matnr m~charg m~bwart m~menge m~aufnr a~matnr
                    FROM mseg AS m
                    INNER JOIN afpo AS a
                    ON a~aufnr = m~aufnr
                    INTO TABLE t_mseg
                    FOR ALL ENTRIES IN t_alternativa
                    WHERE m~matnr = t_alternativa-matnr
                      AND m~charg = t_alternativa-charg
                      AND a~matnr = t_alternativa-matnr_pa
                      AND ( m~bwart = '261' OR m~bwart = '262' ).

                  LOOP AT t_mseg INTO wa_mseg.

                    IF wa_mseg-bwart = '262'.
                      wa_mseg-menge = wa_mseg-menge * -1.
                    ENDIF.

                    CLEAR: wa_mseg-bwart, wa_mseg-aufnr, wa_mseg-matnr_apl.

                    COLLECT wa_mseg INTO t_mseg_sum.

                  ENDLOOP.

                ENDIF.

                SORT: lt_pajt_log BY werks matnr charg,
                      t_mseg_sum  BY werks matnr charg matnr_apl.
*PP - Melhorias MP Alternativa - RDM 271191

                "Lote compartilhado
                SELECT *
                 INTO TABLE t_oest_log
                 FROM zppt_oest_log
                 FOR ALL ENTRIES IN t_mchb_alv
                 WHERE matnr    EQ t_mchb_alv-matnr
                   AND charg    EQ t_mchb_alv-charg
                   AND situacao EQ 'APROVADO'.
                SORT t_oest_log BY matnr charg.

                DATA(t_mchb_alv_aux) = t_mchb_alv[].
                SORT t_mchb_alv_aux BY matnr.
                DELETE ADJACENT DUPLICATES FROM t_mchb_alv_aux COMPARING matnr.

                IF t_mchb_alv_aux[] IS NOT INITIAL.
                  SELECT matnr, meins
                    INTO TABLE @DATA(lt_mara)
                      FROM mara
                        FOR ALL ENTRIES IN @t_mchb_alv_aux
                        WHERE matnr = @t_mchb_alv_aux-matnr.
                ENDIF.

                CLEAR: v_tab_aux.
                LOOP AT t_mchb_alv INTO wa_mchb_alv.
                  v_tab_aux = sy-tabix.

                  READ TABLE t_alternativa INTO wa_alternativa WITH KEY matnr = wa_mchb_alv-matnr
                                                                        charg = wa_mchb_alv-charg.
                  wa_mchb_alv-obs   = wa_alternativa-obs.
                  wa_mchb_alv-ersda = wa_alternativa-dt_lib_qa.

*                  READ TABLE t_mseg_sum INTO wa_mseg_sum WITH KEY werks = wa_mchb_alv-werks
*                                                                  matnr = wa_mchb_alv-matnr
*                                                                  charg = wa_mchb_alv-charg BINARY SEARCH.
*                  IF sy-subrc EQ 0.
*                    v_clabs = wa_alternativa-clabs - wa_mseg_sum-menge.
*
*                    IF wa_mchb_alv-clabs < v_clabs.
*                      wa_mchb_alv-clabs = wa_mchb_alv-clabs.
*                    ELSE.
*                      wa_mchb_alv-clabs = v_clabs.
*                      wa_mchb_alv-resbp = 'X'.
*                    ENDIF.
*
*                  ELSE.
*
*                    IF wa_mchb_alv-clabs < wa_alternativa-clabs.
*                      wa_mchb_alv-clabs = wa_mchb_alv-clabs.
*                    ELSE.
*                      wa_mchb_alv-clabs = wa_alternativa-clabs.
*                    ENDIF.
*
*                  ENDIF.
                  CALL FUNCTION 'ZPPF_SALDO_ALTERNATIVO'
                    EXPORTING
                      iv_matnr        = wa_alternativa-matnr
                      iv_charg        = wa_alternativa-charg
                      iv_matnt_apl    = wa_alternativa-matnr_pa
                      iv_rsnum        = wa_alternativa-rsnum
                    IMPORTING
                      ev_saldoest     = lv_saldoest
                      ev_qtde_reserva = lv_qtde_reserva
                      ev_qtde_estoque = lv_qtde_estoque.

                  lv_soma = lv_qtde_reserva + lv_qtde_estoque.
                  wa_mchb_alv-clabs = wa_alternativa-clabs - lv_soma.
                  IF wa_mchb_alv-clabs < 0.
                    wa_mchb_alv-clabs = 0.
                  ENDIF.

                  IF lv_saldoest < wa_mchb_alv-clabs.
                    wa_mchb_alv-clabs = lv_saldoest.
                  ENDIF.

                  TRY.
                      DATA(ls_mara) = lt_mara[ matnr = wa_alternativa-matnr ].
                      IF NOT ls_mara-meins = 'TO'.
                        CALL FUNCTION 'MD_CONVERT_MATERIAL_UNIT'
                          EXPORTING
                            i_matnr              = ls_mara-matnr
                            i_in_me              = ls_mara-meins
                            i_out_me             = 'TO'
                            i_menge              = wa_mchb_alv-clabs
                          IMPORTING
                            e_menge              = wa_mchb_alv-clabs
                          EXCEPTIONS
                            error_in_application = 1
                            error                = 2
                            OTHERS               = 3.
                      ENDIF.
                    CATCH cx_sy_itab_line_not_found.
                  ENDTRY.

                  "Lote compartilhado
                  CLEAR t_oest_log_aux[].
                  t_oest_log_aux[] = t_oest_log[].
                  DELETE t_oest_log_aux WHERE matnr <> wa_mchb_alv-matnr OR charg <> wa_mchb_alv-charg.

                  IF lines( t_oest_log_aux ) = 1 OR t_oest_log_aux IS INITIAL.
                    wa_mchb_alv-lote_compart = space.

                  ELSEIF lines( t_oest_log_aux ) = 2.

                    LOOP AT t_oest_log_aux INTO DATA(wa_oest_log_aux) WHERE matnr_apl <> t_saida-matnr.
                      wa_mchb_alv-lote_compart = wa_oest_log_aux-matnr_apl.
                      wa_mchb_alv-resbp = wa_oest_log_aux-resbp.
                    ENDLOOP.

                  ELSE.
                    wa_mchb_alv-lote_compart = 'VARIOS'.
                  ENDIF.

                  MODIFY t_mchb_alv FROM wa_mchb_alv INDEX v_tab_aux.
                  CLEAR wa_mchb_alv.
                ENDLOOP.

*RDM316390
                DELETE t_mchb_alv WHERE clabs LE 0.

                IF t_mchb_alv[] IS NOT INITIAL.
*RDM316390

                  SORT: t_mchb_alv BY ersda.
                  gt_fieldcat-fieldname = 'MATNR'.
                  gt_fieldcat-seltext_m = 'Material'.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.

                  gt_fieldcat-fieldname = 'WERKS'.
                  gt_fieldcat-seltext_m = 'Centro'.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.

                  gt_fieldcat-fieldname = 'LGORT'.
                  gt_fieldcat-seltext_m = 'Depósito'.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.

                  gt_fieldcat-fieldname = 'CHARG'.
                  gt_fieldcat-seltext_m = 'Lote'.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.

                  gt_fieldcat-fieldname = 'ERSDA'.
                  gt_fieldcat-seltext_m = 'Data de Criação'.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.

                  gt_fieldcat-fieldname = 'CLABS'.
                  gt_fieldcat-seltext_m = 'Peso'.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.

*PP - Melhorias MP Alternativa - RDM 271191
                  gt_fieldcat-fieldname = 'RESBP'.
                  gt_fieldcat-seltext_m = 'Reserva Parcial'.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.
*PP - Melhorias MP Alternativa - RDM 271191

                  gt_fieldcat-fieldname = 'OBS'.
                  gt_fieldcat-seltext_m = 'Observação'.
                  gt_fieldcat-outputlen = 60.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.

                  gt_fieldcat-fieldname = 'LOTE_COMPART'.
                  gt_fieldcat-seltext_m = 'Compartilhado Com'.
                  gt_fieldcat-outputlen = 18.
                  gt_fieldcat-hotspot   = 'X'.
                  APPEND gt_fieldcat.
                  CLEAR: gt_fieldcat.

                  ls_excl-fcode = '&ETA'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '%SC'.  APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '%SC+'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&OUP'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&ODN'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&ILT'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&OL0'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&CRB'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&CRL'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&CRR'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&CRE'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&AC1'. APPEND ls_excl TO lt_excl. " cancel
                  ls_excl-fcode = '&ALL'. APPEND ls_excl TO lt_excl.
                  ls_excl-fcode = '&SAL'. APPEND ls_excl TO lt_excl.

                  lv_repid = sy-repid.

                  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
                    EXPORTING
                      i_grid_title            = 'Relatório Detalhado'
                      i_structure_name        = 'T_MCHB_ALV'
                      it_fieldcat             = gt_fieldcat[]
                      it_excluding            = lt_excl
                      i_callback_program      = lv_repid
                      i_screen_start_column   = 5
                      i_screen_start_line     = 5
                      i_screen_end_column     = 150
                      i_screen_end_line       = 15
                      i_callback_user_command = 'USER_COMMAND'
                    TABLES
                      t_outtab                = t_mchb_alv
                    EXCEPTIONS
                      program_error           = 1
                      OTHERS                  = 2.
                  IF sy-subrc <> 0.
                    MESSAGE i000(0k) WITH sy-subrc.
                  ENDIF.
*RDM316390
                ELSE.
                  MESSAGE 'Não há estoque para esta matéria prima' TYPE 'S' DISPLAY LIKE 'E'.
                  RETURN.
                ENDIF.
*RDM316390
              ELSE."Não encontrou registros
                MESSAGE 'Não há estoque para esta matéria prima' TYPE 'S' DISPLAY LIKE 'E'.
                RETURN.
              ENDIF."Caso a tabela de estoque de lotes(MCHB) não esteja vazia - Fim
            ELSE."Não se encontrou dados
              MESSAGE 'Não há estoque para esta matéria prima' TYPE 'S' DISPLAY LIKE 'E'.
              RETURN.
            ENDIF."Verifica se encontrou dados - Fim

***Inserção - Hermiro Carvalho - RDM 140328 - 22/06/2015 - Fim
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-MATERIA_PRIMA' AND rs_selfield-value IS NOT INITIAL.
*INICIO: PP - ZSUPPR_PLANO_CORTE - RDM 194058

            PERFORM sku USING rs_selfield-value.


            "           SET PARAMETER ID: 'MAT' FIELD rs_selfield-value.
            "           CALL TRANSACTION 'MM03' AND SKIP FIRST SCREEN.
*FIM   : PP - ZSUPPR_PLANO_CORTE - RDM 194058

***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Início
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-OV_MD81'.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            CLEAR: v_vbeln, v_posnr, v_kdmat_aux.
            SELECT SINGLE a~vbeln a~posnr a~kdmat
              INTO (v_vbeln, v_posnr, v_kdmat_aux)
              FROM vbap AS a INNER JOIN vbak AS b
                ON ( a~vbeln = b~vbeln )
             WHERE a~matnr = t_saida-matnr  AND
                   a~kdmat = t_saida-codigo AND
                   a~werks = t_saida-werks  AND
                   b~auart = '01'.
            SET PARAMETER ID: 'AUN' FIELD v_vbeln.
            CALL TRANSACTION 'MD83' AND SKIP FIRST SCREEN.
***Inserção - Hermiro Carvalho - PRJ01313 - 16/09/2015 - Fim
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-PN_INTERNO' AND rs_selfield-value IS NOT INITIAL.
            SET PARAMETER ID 'ZPN' FIELD rs_selfield-value.
            CALL TRANSACTION 'ZQM_PN3' AND SKIP FIRST SCREEN.
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Início
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_FORN'.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            CLEAR: t_mslb_log[], lt_fcat[], ls_fcat, gt_fieldcat[],
                   wa_mslb_log, lt_excl[], ls_excl.
***Inserção - Sidney Q de Andrade - INC 2165524 - 13/07/2016 - Início
*            SELECT matnr werks charg lifnr lblab
*              INTO CORRESPONDING FIELDS OF TABLE t_mslb_log
*              FROM mslb
*             WHERE werks = t_saida-werks
*               AND matnr = t_saida-materia_prima.
*            CASE sy-subrc."Verifica se encontrou registros - Início
            LOOP AT t_mslb INTO wa_mslb_log WHERE werks = t_saida-werks AND
                                                  matnr = t_saida-materia_prima.
              APPEND wa_mslb_log TO t_mslb_log.
              CLEAR wa_mslb_log.
            ENDLOOP.

            IF t_mslb_log[] IS NOT INITIAL.
***Inserção - Sidney Q de Andrade - INC 2165524 - 13/07/2016 - Fim
              SORT: t_mslb_log BY werks charg lifnr.

              FREE: gt_fieldcat_i.
              CLEAR: gs_layout_i, gt_fieldcat_i.

              gs_layout_i-box_tabname          = 'T_MSLB_LOG'.
              gs_layout_i-zebra                = 'X'.

              gt_fieldcat_i-fieldname = 'WERKS'.
              gt_fieldcat_i-seltext_m = 'Centro'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              gt_fieldcat_i-fieldname = 'MATNR'.
              gt_fieldcat_i-seltext_m = 'Material'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              gt_fieldcat_i-fieldname = 'CHARG'.
              gt_fieldcat_i-hotspot   = 'X'.
              gt_fieldcat_i-seltext_m = 'Lote'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              gt_fieldcat_i-fieldname = 'LIFNR'.
              gt_fieldcat_i-seltext_m = 'Fornecedor'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              gt_fieldcat_i-fieldname = 'LBLAB'.
              gt_fieldcat_i-seltext_m = 'Estoque'.
              gt_fieldcat_i-do_sum    = 'X'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
                EXPORTING
                  i_callback_program       = sy-repid
                  is_layout                = gs_layout_i
                  it_fieldcat              = gt_fieldcat_i[]
                  i_callback_user_command  = 'F_USER_COMMAND_I'
                  i_callback_pf_status_set = 'F_PF_STATUS_SET_I'
                  i_default                = 'X'
                TABLES
                  t_outtab                 = t_mslb_log
                EXCEPTIONS
                  program_error            = 1
                  OTHERS                   = 2.
              IF sy-subrc <> 0.
                MESSAGE i000(0k) WITH sy-subrc.
              ENDIF.
***Inserção - Sidney Q de Andrade - INC 2165524 - 13/07/2016 - Inicio
            ENDIF.
*            ENDCASE."Verifica se encontrou registros - Fim
***Inserção - Sidney Q de Andrade - INC 2165524 - 13/07/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Início
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_FORN_LTAN'.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            CLEAR: t_mslb_log[], lt_fcat[], ls_fcat, gt_fieldcat[],
                   wa_mslb_log, lt_excl[], ls_excl.
            LOOP AT t_hist_lt INTO wa_hist_lt WHERE mantr = t_saida-matnr AND
                                                    werks = t_saida-werks.
              LOOP AT t_mslb_ltan INTO wa_mslb_log WHERE werks = wa_hist_lt-werks AND
                                                         matnr = wa_hist_lt-idnrk.
                APPEND wa_mslb_log TO t_mslb_log.
                CLEAR wa_mslb_log.
              ENDLOOP.
            ENDLOOP.
            IF t_mslb_log[] IS NOT INITIAL.

              SORT: t_mslb_log BY werks charg lifnr.

              FREE: gt_fieldcat_i.
              CLEAR: gs_layout_i, gt_fieldcat_i.

              gs_layout_i-box_tabname          = 'T_MSLB_LOG'.
              gs_layout_i-zebra                = 'X'.

              gt_fieldcat_i-fieldname = 'WERKS'.
              gt_fieldcat_i-seltext_m = 'Centro'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              gt_fieldcat_i-fieldname = 'MATNR'.
              gt_fieldcat_i-seltext_m = 'Material'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              gt_fieldcat_i-fieldname = 'CHARG'.
              gt_fieldcat_i-hotspot   = 'X'.
              gt_fieldcat_i-seltext_m = 'Lote'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              gt_fieldcat_i-fieldname = 'LIFNR'.
              gt_fieldcat_i-seltext_m = 'Fornecedor'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              gt_fieldcat_i-fieldname = 'LBLAB'.
              gt_fieldcat_i-seltext_m = 'Estoque'.
              gt_fieldcat_i-do_sum    = 'X'.
              APPEND gt_fieldcat_i.
              CLEAR: gt_fieldcat_i.

              CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
                EXPORTING
                  i_callback_program       = sy-repid
                  is_layout                = gs_layout_i
                  it_fieldcat              = gt_fieldcat_i[]
                  i_callback_user_command  = 'F_USER_COMMAND_I'
                  i_callback_pf_status_set = 'F_PF_STATUS_SET_I'
                  i_default                = 'X'
                TABLES
                  t_outtab                 = t_mslb_log
                EXCEPTIONS
                  program_error            = 1
                  OTHERS                   = 2.
              IF sy-subrc <> 0.
                MESSAGE i000(0k) WITH sy-subrc.
              ENDIF.
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Fim
***Inserção - Hermiro Carvalho - RDM 185165 - 10/08/2017 - Início
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_MP'.
            CLEAR: t_saida.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            CASE sy-subrc.
              WHEN 0.
***Inserção - Hermiro Carvalho - RDM 186042 - 18/08/2017 - Início
*                SET PARAMETER ID: 'WRK' FIELD t_saida-werks.
*                SET PARAMETER ID: 'MAT' FIELD t_saida-materia_prima.
*                CALL TRANSACTION 'MMBE' AND SKIP FIRST SCREEN.
***Inserção - Hermiro Carvalho - RDM 186735 - 19/09/2017 - Início
                FREE: t_bdc,
                      t_return_bdc.
                IF t_saida-estoque_mp IS NOT INITIAL.
***Inserção - Hermiro Carvalho - RDM 186735 - 26/09/2017 - Início
                  CLEAR: vl_option.
                  PERFORM verifica_opcao CHANGING vl_option.
                  CASE vl_option."Verifica opção do usuário
                    WHEN 1."MMBE
                      PERFORM insere_bdc USING 'X' 'RMMMBESTN'          '1000'.
                      PERFORM insere_bdc USING ''  'BDC_CURSOR'         'MS_WERKS-LOW'.
                      PERFORM insere_bdc USING ''  'BDC_OKCODE'         '=ONLI'.
                      PERFORM insere_bdc USING ''  'MS_MATNR-LOW'       t_saida-materia_prima.
                      PERFORM insere_bdc USING ''  'MS_WERKS-LOW'       space.
                      PERFORM insere_bdc USING ''  'KZLSO'              'X'.
                      PERFORM insere_bdc USING ''  'KZLON'              'X'.
                      PERFORM insere_bdc USING ''  'VERNU'              '1'.
                      PERFORM insere_bdc USING ''  'KZNUL'              'X'.
                      PERFORM insere_bdc USING ''  'KZBUK'              'X'.
                      PERFORM insere_bdc USING ''  'KZWER'              'X'.
                      PERFORM insere_bdc USING ''  'KZLGO'              'X'.
                      PERFORM insere_bdc USING ''  'KZCHA'              'X'.
                      PERFORM insere_bdc USING ''  'KZSON'              'X'.
*                      CASE sy-uname.
*                        WHEN 'RGC4691' OR 'RG01266'.
*                          wa_opt-dismode = 'A'.
*                          wa_opt-defsize = 8.
*                        WHEN OTHERS.
                      wa_opt-dismode = 'E'.
                      wa_opt-defsize = 8.
*                      ENDCASE.
                      SET PARAMETER ID: 'WRK' FIELD space.
                      CALL TRANSACTION 'MMBE' USING t_bdc OPTIONS FROM wa_opt.
                    WHEN 2."ZA09A
                      CLEAR: seltab.
                      seltab_wa-selname = 'P_WERKS'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'BT'.
                      seltab_wa-low     = 'AB24'.
                      seltab_wa-high    = 'ZZZZ'.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.

                      seltab_wa-selname = 'SO_LGORT'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'BT'.
                      seltab_wa-low     = '0010'.
                      seltab_wa-high    = '0999'.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.

                      seltab_wa-selname = 'P_MATE'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'EQ'.
                      seltab_wa-low     = t_saida-materia_mp.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.
                      SUBMIT zrnsdr_mat_pri_est_rn WITH p_todos = 'X'
                                                   WITH p_alv   = 'X'
                                                   WITH SELECTION-TABLE seltab AND RETURN.
                    WHEN OTHERS.
                  ENDCASE.
                ENDIF.
***Inserção - Hermiro Carvalho - RDM 186735 - 26/09/2017 - Fim
              WHEN OTHERS.
            ENDCASE.

          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_MP_OUTRAS'.
            CLEAR: t_saida.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            CASE sy-subrc.
              WHEN 0.
                FREE: t_bdc,
                      t_return_bdc.
                IF t_saida-estoque_mp_outras IS NOT INITIAL.
                  CLEAR: vl_option.
                  PERFORM verifica_opcao CHANGING vl_option.
                  CASE vl_option."Verifica opção do usuário
                    WHEN 1."MMBE
                      PERFORM insere_bdc USING 'X' 'RMMMBESTN'          '1000'.
                      PERFORM insere_bdc USING ''  'BDC_CURSOR'         'MS_WERKS-LOW'.
                      PERFORM insere_bdc USING ''  'BDC_OKCODE'         '=ONLI'.
                      PERFORM insere_bdc USING ''  'MS_MATNR-LOW'       t_saida-materia_prima.
                      PERFORM insere_bdc USING ''  'MS_WERKS-LOW'       space.
                      PERFORM insere_bdc USING ''  'KZLSO'              'X'.
                      PERFORM insere_bdc USING ''  'KZLON'              'X'.
                      PERFORM insere_bdc USING ''  'VERNU'              '1'.
                      PERFORM insere_bdc USING ''  'KZNUL'              'X'.
                      PERFORM insere_bdc USING ''  'KZBUK'              'X'.
                      PERFORM insere_bdc USING ''  'KZWER'              'X'.
                      PERFORM insere_bdc USING ''  'KZLGO'              'X'.
                      PERFORM insere_bdc USING ''  'KZCHA'              'X'.
                      PERFORM insere_bdc USING ''  'KZSON'              'X'.
                      wa_opt-dismode = 'E'.
                      wa_opt-defsize = 8.
                      SET PARAMETER ID: 'WRK' FIELD space.
                      CALL TRANSACTION 'MMBE' USING t_bdc OPTIONS FROM wa_opt.
                    WHEN 2."ZA09A
                      CLEAR: seltab.
                      seltab_wa-selname = 'P_WERKS'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'BT'.
                      seltab_wa-low     = 'AB24'.
                      seltab_wa-high    = 'ZZZZ'.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.

                      seltab_wa-selname = 'SO_LGORT'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'BT'.
                      seltab_wa-low     = '0010'.
                      seltab_wa-high    = '0999'.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.

                      seltab_wa-selname = 'P_MATE'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'EQ'.
                      seltab_wa-low     = t_saida-materia_mp.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.
                      SUBMIT zrnsdr_mat_pri_est_rn WITH p_todos = 'X'
                                                   WITH p_alv   = 'X'
                                                   WITH SELECTION-TABLE seltab AND RETURN.
                    WHEN OTHERS.
                  ENDCASE.
                ENDIF.
              WHEN OTHERS.
            ENDCASE.

          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_MP_CEN'.
            CLEAR: t_saida.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            CASE sy-subrc.
              WHEN 0.
*                SET PARAMETER ID: 'MAT' FIELD t_saida-materia_prima.
*                CALL TRANSACTION 'MMBE' AND SKIP FIRST SCREEN.
                FREE: t_bdc,
                      t_return_bdc.
                IF t_saida-estoque_mp_cen IS NOT INITIAL.
***Inserção - Hermiro Carvalho - RDM 186735 - 19/09/2017 - Início
***Inserção - Hermiro Carvalho - RDM 186735 - 26/09/2017 - Início
                  CLEAR: vl_option.
                  PERFORM verifica_opcao CHANGING vl_option.
                  CASE vl_option."Verifica opção do usuário
                    WHEN 1."MMBE
***Inserção - Hermiro Carvalho - RDM 186735 - 27/09/2017 - Início
                      PERFORM insere_bdc USING 'X' 'RMMMBESTN'          '1000'.
                      PERFORM insere_bdc USING ''  'BDC_CURSOR'         'MS_WERKS-LOW'.
*                      PERFORM insere_bdc USING ''  'BDC_OKCODE'         '=ONLI'.
                      PERFORM insere_bdc USING ''  'BDC_OKCODE'         '=%002'.
                      PERFORM insere_bdc USING ''  'MS_MATNR-LOW'       t_saida-materia_prima.
                      PERFORM insere_bdc USING ''  'MS_WERKS-LOW'       t_saida-werks.
                      PERFORM insere_bdc USING ''  'KZLSO'              'X'.
                      PERFORM insere_bdc USING ''  'KZLON'              'X'.
                      PERFORM insere_bdc USING ''  'VERNU'              '1'.
                      PERFORM insere_bdc USING ''  'KZNUL'              'X'.
                      PERFORM insere_bdc USING ''  'KZBUK'              'X'.
                      PERFORM insere_bdc USING ''  'KZWER'              'X'.
                      PERFORM insere_bdc USING ''  'KZLGO'              'X'.
                      PERFORM insere_bdc USING ''  'KZCHA'              'X'.
                      PERFORM insere_bdc USING ''  'KZSON'              'X'.
***Comentar - Hermiro Carvalho - RDM 186735 - 28/09/2017 - Início
***Faz a MMBE trabalhar com um centro diferente do indicado no batch iniput
                      PERFORM insere_bdc USING 'X' 'SAPLALDB'          '3000'.
                      PERFORM insere_bdc USING ''  'BDC_OKCODE'        '=SI01'.
                      PERFORM insere_bdc USING ''  'BDC_SUBSCR'        'SAPLALDB                                3010SCREEN_HEADER'.
                      PERFORM insere_bdc USING ''  'BDC_CURSOR'        'RSCSEL_255-SOP_I(01)'.
                      PERFORM insere_bdc USING ''  'RSCSEL_255-SLOW_I(01)'        t_saida-werks.

                      PERFORM insere_bdc USING 'X' 'SAPMSSY0'           '0120'.
                      PERFORM insere_bdc USING ''  'BDC_CURSOR'         '11/11'.
                      PERFORM insere_bdc USING ''  'BDC_OKCODE'         '=PICK'.

                      PERFORM insere_bdc USING 'X' 'SAPMSSY0'           '0120'.
                      PERFORM insere_bdc USING ''  'BDC_CURSOR'         '11/09'.
                      PERFORM insere_bdc USING ''  'BDC_OKCODE'         '=OKAY'.

                      PERFORM insere_bdc USING 'X' 'SAPLALDB'          '3000'.
                      PERFORM insere_bdc USING ''  'BDC_OKCODE'        '=ACPT'.
                      PERFORM insere_bdc USING ''  'BDC_SUBSCR'        'SAPLALDB                                3010SCREEN_HEADER'.
                      PERFORM insere_bdc USING ''  'BDC_CURSOR'        'RSCSEL_255-SOP_I(01)'.
                      PERFORM insere_bdc USING ''  'RSCSEL_255-SLOW_I(01)'        t_saida-werks.


                      PERFORM insere_bdc USING 'X' 'RMMMBESTN'          '1000'.
                      PERFORM insere_bdc USING ''  'BDC_CURSOR'         'MS_WERKS-LOW'.
                      PERFORM insere_bdc USING ''  'BDC_OKCODE'         '=ONLI'.
                      PERFORM insere_bdc USING ''  'MS_MATNR-LOW'       t_saida-materia_prima.
                      PERFORM insere_bdc USING ''  'MS_WERKS-LOW'       t_saida-werks.
                      PERFORM insere_bdc USING ''  'KZLSO'              'X'.
                      PERFORM insere_bdc USING ''  'KZLON'              'X'.
                      PERFORM insere_bdc USING ''  'VERNU'              '1'.
                      PERFORM insere_bdc USING ''  'KZNUL'              'X'.
                      PERFORM insere_bdc USING ''  'KZBUK'              'X'.
                      PERFORM insere_bdc USING ''  'KZWER'              'X'.
                      PERFORM insere_bdc USING ''  'KZLGO'              'X'.
                      PERFORM insere_bdc USING ''  'KZCHA'              'X'.
                      PERFORM insere_bdc USING ''  'KZSON'              'X'.
***Comentar - Hermiro Carvalho - RDM 186735 - 28/09/2017 - Fim
                      CASE sy-uname.
                        WHEN 'RGC4691' OR 'RG01266' OR 'RGC8096'.
                          wa_opt-dismode = 'A'.
                          wa_opt-defsize = 8.
                        WHEN OTHERS.
                          wa_opt-dismode = 'E'.
                          wa_opt-defsize = 8.
                      ENDCASE.
*                      SET PARAMETER ID: 'MAT' FIELD t_saida-materia_prima.
*                      SET PARAMETER ID: 'WRK' FIELD space.
*                      CALL TRANSACTION 'MMBE' AND SKIP FIRST SCREEN.
                      CALL TRANSACTION 'MMBE' USING t_bdc OPTIONS FROM wa_opt.
***Inserção - Hermiro Carvalho - RDM 186735 - 27/09/2017 - Fim
                    WHEN 2."ZA09A
                      CLEAR: seltab.
                      seltab_wa-selname = 'P_WERKS'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'NE'.
                      seltab_wa-low     = t_saida-werks.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.

                      seltab_wa-selname = 'SO_LGORT'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'BT'.
                      seltab_wa-low     = '0010'.
                      seltab_wa-high    = '0999'.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.

                      seltab_wa-selname = 'P_MATE'.
                      seltab_wa-sign    = 'I'.
                      seltab_wa-option  = 'EQ'.
                      seltab_wa-low     = t_saida-materia_mp.
                      APPEND seltab_wa TO seltab.
                      CLEAR: seltab_wa.
                      SUBMIT zrnsdr_mat_pri_est_rn WITH p_todos = 'X'
                                                   WITH p_alv   = 'X'
                                                   WITH SELECTION-TABLE seltab AND RETURN.
                    WHEN OTHERS.
                  ENDCASE.
***Inserção - Hermiro Carvalho - RDM 186735 - 19/09/2017 - Fim
***Inserção - Hermiro Carvalho - RDM 186735 - 26/09/2017 - Fim
                ENDIF.
***Inserção - Hermiro Carvalho - RDM 186042 - 18/08/2017 - Fim
              WHEN OTHERS.
            ENDCASE.
***Inserção - Hermiro Carvalho - RDM 185165 - 10/08/2017 - Fim
***Inserção - Hermiro Carvalho - RDM 187497 - 21/11/2017 - Início
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-PREV_ORG'.
            IF t_saida-matnr BETWEEN '34000000' AND '34999999'.
              PERFORM visualiza_info_mapa USING t_saida-werks t_saida-materia_prima.
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 187497 - 21/11/2017 - Fim
***Inserção - Hermiro Carvalho - RDM 196432 - 05/04/2018 - Início
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-CALL_OBS'.
            IF t_saida-matnr IS INITIAL OR p_period-low IS INITIAL.
              MESSAGE 'Favor, selecione uma linha com dados válidos' TYPE 'I'.
              MESSAGE 'Favor, selecione uma linha com dados válidos' TYPE 'S' DISPLAY LIKE 'E'.
              RETURN.
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 196432 - 20/04/2018 - Início
            CLEAR: vl_erro, vl_retorno.
            PERFORM visualiza_observacao_gd USING t_saida-werks t_saida-matnr p_period-low p_histob
                                         CHANGING vl_erro vl_retorno.
            CLEAR: t_saida-observacao, lv_ref_grid.
            PERFORM verifica_primeira_linha USING t_saida-werks t_saida-matnr p_period-low CHANGING t_saida-observacao.
            MODIFY t_saida INDEX rs_selfield-tabindex.
            IF lv_ref_grid IS INITIAL.
              CALL FUNCTION 'GET_GLOBALS_FROM_SLVC_FULLSCR'
                IMPORTING
                  e_grid = lv_ref_grid.
            ENDIF.
            IF lv_ref_grid IS NOT INITIAL.
              CALL METHOD lv_ref_grid->check_changed_data.
*              CALL METHOD lv_ref_grid->refresh_table_display.
              rs_selfield-refresh = 'X'.
            ENDIF.
            IF vl_erro IS NOT INITIAL."Após o refresh do ALV, se houver erro, mostra a mensagem - Início
              MESSAGE vl_retorno TYPE 'I'.
              MESSAGE vl_retorno TYPE 'S' DISPLAY LIKE 'W'.
            ENDIF."Após o refresh do ALV, se houver erro, mostra a mensagem - Fim
***Inserção - Hermiro Carvalho - RDM 196432 - 20/04/2018 - Fim
***Inserção - Hermiro Carvalho - RDM 196432 - 05/04/2018 - Fim
***Inserção - Hermiro Carvalho - RDM 196641 - 10/08/2018 - Início
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-S_MP'.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            IF t_saida-s_mp NE '0'.
              PERFORM alv_popup_select USING 'S_MP' 'SINTET' rs_selfield.
            ENDIF.
***Inserção - Hermiro Carvalho - RDM 196641 - 10/08/2018 - Fim

* >>> - 10.10.2019 11:33:31 - RDM 225838 - Simone
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTQTRAN' AND rs_selfield-value IS NOT INITIAL.
            PERFORM alv_estoque_transito_i.
* <<< - 10.10.2019 13:33:31 - RDM 225838 - Simone

***Inicio - Fabio - 19.08.2020 - RDM 238614
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-VOLUME_ICON' AND rs_selfield-value IS NOT INITIAL.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            IF t_saida-volume_rem IS NOT INITIAL.
              PERFORM alv_popup_select_volume.
            ENDIF.
***Fim - Fabio - 19.08.2020 - RDM 238614

*PP - Cenário Estoque Consignado - RDM 239896
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTOQUE_CONSIGNADO' AND rs_selfield-value IS NOT INITIAL.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            IF sy-subrc IS INITIAL.
              PERFORM alv_estoque_consignado.
            ENDIF.
*PP - Cenário Estoque Consignado - RDM 239896
          ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-QTD_MP_CHE' AND rs_selfield-value IS NOT INITIAL.
            READ TABLE t_saida INDEX rs_selfield-tabindex.
            IF sy-subrc IS INITIAL.
              PERFORM f_detalhe_clge USING rs_selfield-tabindex.
            ENDIF.

          ENDIF.
        ENDIF.
    ENDCASE.
  ENDIF.

***Inserção - Hermiro Carvalho - RDM 196641 - 09/08/2018 - Início
  IF op1  = 'X'.
    READ TABLE t_saida INDEX rs_selfield-tabindex.
    IF rs_selfield-sel_tab_field = 'T_SAIDA-S_MP' AND t_saida-s_mp NE '0'.
      PERFORM alv_popup_select USING 'S_MP' 'SINTET' rs_selfield.

* >>> - 10.10.2019 11:33:31 - RDM 225838 - Simone
    ELSEIF rs_selfield-sel_tab_field = 'T_SAIDA-ESTQTRAN' AND rs_selfield-value IS NOT INITIAL.
      PERFORM alv_estoque_transito_i.
* <<< - 10.10.2019 13:33:31 - RDM 225838 - Simone
    ENDIF.
  ENDIF.
***Inserção - Hermiro Carvalho - RDM 196641 - 09/08/2018 - Fim
ENDFORM.                    "F_USER_COMMAND
*&---------------------------------------------------------------------*
*&      Form  INSERE_BDC
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_2306   Tela
*      -->P_2307   Campo
*      -->P_2308   Valor
*----------------------------------------------------------------------*
FORM insere_bdc  USING    VALUE(p_2306)
                          VALUE(p_2307)
                          VALUE(p_2308).
  CLEAR wa_bdc.
  IF p_2306 EQ 'X'.
    wa_bdc-dynbegin  = 'X'.
    wa_bdc-program   = p_2307.
    wa_bdc-dynpro    = p_2308.
  ELSE.
    wa_bdc-fnam      = p_2307.
    wa_bdc-fval      = p_2308.
  ENDIF.
  APPEND wa_bdc TO t_bdc.


ENDFORM.                    " INSERE_BDC
*&---------------------------------------------------------------------*
*&      Form  GET_STOCK_DATE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      <--P_T_MCHB_ALV_ERSDA  text
*----------------------------------------------------------------------*
FORM get_stock_date  CHANGING date TYPE datum.

  DATA doc_id TYPE mblnr.
  DATA rowcount TYPE d.

  SELECT mkpf~mblnr bldat
    FROM mseg
    INNER JOIN mkpf
    ON mkpf~mblnr = mseg~mblnr
    AND mkpf~mjahr = mseg~mjahr
    INTO (doc_id, date)
    WHERE ( mseg~bwart = '101'
    OR mseg~bwart = '105'
    OR mseg~bwart = '913'
    OR mseg~bwart = '911'
    OR mseg~bwart = '309'
    OR mseg~bwart = '561'
    OR mseg~bwart = '861' )
*    AND mseg~matnr = wa_mchb-matnr
*    AND mseg~werks = wa_mchb-werks
*    AND mseg~lgort = wa_mchb-lgort
    AND mseg~charg = wa_mchb_alv-charg.

    SELECT SINGLE COUNT(*)
        FROM mseg
        INTO rowcount
        WHERE mseg~smbln = doc_id.

    IF rowcount = 0.
      EXIT.
    ENDIF.
  ENDSELECT.


ENDFORM.                    " GET_STOCK_DATE
*&---------------------------------------------------------------------*
*&      Form  PLANO_DE_CORTE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM plano_de_corte .

  DATA: v_werks       TYPE vbap-werks,
        v_producao(9) TYPE c,
        v_num_seq     TYPE char10,
        v_plan_name   TYPE char30,
        v_vbeln       TYPE char15,
        v_posnr       TYPE char15,
        v_count       TYPE i,
        v_numero(2)   TYPE c,
        v_ovmd81(17)  TYPE c,
        v_kzkfg       TYPE mara-kzkfg,
        v_kzkfg_aux   TYPE mara-kzkfg,
        wa_opt        TYPE ctu_params,
        vl_tabix      TYPE sy-tabix,
        v_tabix_c     TYPE string,
        v_cont_aux    TYPE i,
        vl_string     TYPE string,
        v_stock_local TYPE c.

  DATA: t_saida_ord LIKE TABLE OF t_saida WITH HEADER LINE,
        t_vbap      TYPE TABLE OF vbap WITH HEADER LINE.


  READ TABLE t_saida WITH KEY mark = 'X'.
  IF sy-subrc <> 0.
    MESSAGE 'Nenhuma linha selecionada' TYPE 'E'.
  ENDIF.

***Inserção - Hermiro Carvalho - PRJ01313 - 01/02/2016 - Início
  PERFORM z_separa_selecionados CHANGING v_sobsl.
***Inserção - Hermiro Carvalho - PRJ01313 - 01/02/2016 - Fim

  CLEAR: v_werks,
         v_kzkfg,
         v_kzkfg_aux,
         v_resp,
         t_saida_ord,
         t_saida.

  FREE: t_saida_ord.

  LOOP AT t_saida WHERE mark = 'X'.

    IF v_werks NE t_saida-werks AND v_werks IS NOT INITIAL.
      MESSAGE 'Não é permitido selecionar CENTROS distintos'  TYPE 'I'.
      RETURN.
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 159510 - 15/03/2016 - Início
    IF v_producao NE t_saida-producao AND v_producao IS NOT INITIAL.
      MESSAGE 'Não é permitido selecionar Tipos de Produção distintos' TYPE 'I'.
      RETURN.
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 159510 - 15/03/2016 - Fim
    SELECT SINGLE kzkfg
      FROM mara
      INTO v_kzkfg
      WHERE matnr = t_saida-matnr.
***Inserção - Hermiro Carvalho - RDM 159510 - 14/03/2016 - Início
*    IF v_kzkfg NE v_kzkfg_aux AND v_kzkfg_aux IS NOT INITIAL.
*      MESSAGE 'Não é permitido selecionar Materiais com Tipo de Configuração distintas' TYPE 'E'.
*    ENDIF.
***Inserção - Hermiro Carvalho - RDM 159510 - 14/03/2016 - Fim
    MOVE-CORRESPONDING t_saida TO t_saida_ord.
    APPEND t_saida_ord.

    v_werks     = t_saida-werks.
    v_kzkfg_aux = v_kzkfg.
***Inserção - Hermiro Carvalho - RDM 159510 - 15/03/2016 - Início
    v_producao = t_saida-producao.
***Inserção - Hermiro Carvalho - RDM 159510 - 15/03/2016 - Fim
  ENDLOOP.
  CLEAR: t_vbap[], t_plaf_lote[], v_stock_local.
***Inserção - Hermiro Carvalho - RDM 158714 - 29/02/2016 - Início
  IF t_saida-producao = 'EXTERNA'. " exibir popup para o usuário escolher o local do estoque: interno / externo
    CLEAR: v_stock_local.
***Inserção - Hermiro Carvalho - RDM 162929 - 23/05/2016 - Início
    CLEAR: lc_return, lt_fields[], ls_fields.
* Tabela
    ls_fields-tabname   = 'LFA1'.

* Campo Fornecedor
    ls_fields-fieldname = 'LIFNR'.
    ls_fields-field_obl = 'X'.
    APPEND ls_fields TO lt_fields.

    CALL FUNCTION 'POPUP_GET_VALUES'
      EXPORTING
        popup_title     = 'Indique um fornecedor'
        start_column    = '5'
        start_row       = '5'
      IMPORTING
        returncode      = lc_return
      TABLES
        fields          = lt_fields
      EXCEPTIONS
        error_in_fields = 1
        OTHERS          = 2.

    CASE lc_return."Verifica a opção do usuário - Início
      WHEN 'A'."Cancelar
        RETURN.
      WHEN OTHERS."Qualquer outra opção.
        CLEAR: v_fornecedor.
        READ TABLE lt_fields INTO ls_fields INDEX 1.
        SELECT SINGLE lifnr
          INTO v_fornecedor
          FROM lfa1
         WHERE lifnr = ls_fields-value.
        CASE sy-subrc.
          WHEN 0.
          WHEN OTHERS.
            MESSAGE 'Este fornecedor não existe na base de dados!' TYPE 'I'.
            RETURN.
        ENDCASE.
    ENDCASE."Verifica a opção do usuário - Fim
***Inserção - Hermiro Carvalho - RDM 162929 - 23/05/2016 - Fim
    CALL FUNCTION 'POPUP_TO_CONFIRM'
      EXPORTING
        titlebar              = 'Local do Estoque'
        text_question         = 'Selecione o Local do estoque desejado:'
        text_button_1         = 'Interno'
        text_button_2         = 'Externo'
        default_button        = '1'
        display_cancel_button = 'X'
      IMPORTING
        answer                = v_stock_local.
    IF v_stock_local EQ 'A'.
      EXIT.
    ENDIF.
  ELSE.
    v_stock_local = 1.
  ENDIF.

  CLEAR: t_mostra_lotes[], v_string.
  FREE: t_zgr_std_paramtable.
  CLEAR: t_zgr_std_paramtable.
  FREE: t_mostra_lotes.
  CLEAR: t_mostra_lotes[], t_materiais_alv[], t_mostra_lotes_sel[], t_ordens_find[],
         t_mostra_lotes, wa_mate_alv, wa_orve_find, t_ordens_find.
  WHILE t_mostra_lotes_sel[] IS INITIAL.
    CLEAR: t_materiais_alv[], t_ordens_find[], v_string.
    LOOP AT t_saida_ord.
      PERFORM popula_tabelas_lotes_alv USING t_saida_ord.
    ENDLOOP.

    IF t_materiais_alv[] IS NOT INITIAL.
      CALL FUNCTION 'ZPPF_MOSTRA_LOTES_ALV'
        EXPORTING
          werks               = t_saida-werks
          fornecedor          = v_fornecedor
          producao            = v_stock_local
          materiais           = t_materiais_alv[]
          coluna_in           = 10
          line_in             = 5
        IMPORTING
          t_mostra_lotes_out  = t_mostra_lotes_sel[]
          t_mostra_lotes_find = t_ordens_find[]
          mensagem            = v_string.
    ENDIF.

    IF v_string IS NOT INITIAL.
      RETURN."Sai da rotina de pesquisa de lote.
    ENDIF.
  ENDWHILE."Obriga o usuário e selecionar algum lote ou cancelar - Fim
***Inserção - Hermiro Carvalho - RDM 158714 - 08/03/2016 - Início
***Inserção - Hermiro Carvalho - RDM 196353 - 25/05/2018 - Início
***Limpa os memory's ID - Início
  FREE MEMORY ID 'T_LOTES_9040'.
  FREE MEMORY ID 'T_LOTES_9040_AUX'.
  FREE MEMORY ID 'T_LOTES_WERKS'.
  FREE MEMORY ID 'T_LOTES_WERKS_AUX'.
***Limpa os memory's ID - Fim
  IF t_mostra_lotes_sel[] IS NOT INITIAL AND t_ordens_find[] IS NOT INITIAL."Popula as tabelas para o ZPCLORTE
***Lotes selecionados na função
    LOOP AT t_mostra_lotes_sel.
      CLEAR: wa_lotes_prevp.
      wa_lotes_prevp-charg        = t_mostra_lotes_sel-charg_mp.
      wa_lotes_prevp-data_estoque = t_mostra_lotes_sel-data_estoque.
      APPEND wa_lotes_prevp TO t_lotes_ovs.
      wa_lotes_ovs_werks-werks        = t_mostra_lotes_sel-werks.
      wa_lotes_ovs_werks-charg        = t_mostra_lotes_sel-charg_mp.
      wa_lotes_ovs_werks-data_estoque = t_mostra_lotes_sel-data_estoque.
      APPEND wa_lotes_ovs_werks TO t_lotes_ovs_werks.
      CLEAR: wa_lotes_prevp, t_mostra_lotes_sel, wa_lotes_ovs_werks.
    ENDLOOP.

***Lotes encontrados na função.
    LOOP AT t_ordens_find INTO wa_orve_find.
      CLEAR: wa_lotes_prevp.
      wa_lotes_prevp-charg        = wa_orve_find-charg_mp.
      wa_lotes_prevp-data_estoque = wa_orve_find-data_estoque.
      APPEND wa_lotes_prevp TO t_lotes_ovs_aux.
      wa_lotes_ovs_werks_aux-werks        = wa_orve_find-werks.
      wa_lotes_ovs_werks_aux-charg        = wa_orve_find-charg_mp.
      wa_lotes_ovs_werks_aux-data_estoque = wa_orve_find-data_estoque.
      APPEND wa_lotes_ovs_werks_aux TO t_lotes_ovs_werks_aux.
      PERFORM popula_mostra_lotes USING wa_orve_find."Popula a tabela t_mostra_lotes
      CLEAR: wa_lotes_prevp, wa_orve_find, wa_lotes_ovs_werks_aux.
    ENDLOOP.

    EXPORT t_lotes_ovs            TO MEMORY ID 'T_LOTES_9040'.
    EXPORT t_lotes_ovs_aux        TO MEMORY ID 'T_LOTES_9040_AUX'.
    EXPORT t_lotes_ovs_werks      TO MEMORY ID 'T_LOTES_WERKS'.
    EXPORT t_lotes_ovs_werks_aux  TO MEMORY ID 'T_LOTES_WERKS_AUX'.
  ENDIF.

***Inserção - Hermiro Carvalho - RDM 196353 - 25/05/2018 - Fim
*  ENDCASE.
***Inserção - Hermiro Carvalho - RDM 158714 - 08/03/2016 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 11/01/2016 - Fim
  IF t_mostra_lotes[] IS INITIAL.
***Inserção - Hermiro Carvalho - PRJ01313 - 11/01/2016 - Início
    CALL FUNCTION 'POPUP_TO_CONFIRM'
      EXPORTING
        titlebar              = 'Continuar Processo'
        text_question         = 'Não foram encontrados Lotes válidos. Deseja Continuar ?'
        text_button_1         = 'Sim'
        text_button_2         = 'Não'
        default_button        = '1'
        display_cancel_button = 'X'
      IMPORTING
        answer                = v_resp.
    IF sy-subrc <> 0.
* Implement suitable error handling here
    ENDIF.
    IF v_resp EQ 'A' OR v_resp EQ '2'.
      EXIT.
    ENDIF.
  ENDIF.
  IF v_sobsl = c_30.
    vl_obj_name = 'ZPLC_BEXT'.
    v_pl        = 'BEXT'.
    v_plan_name = 'BEXT%99999'.
  ELSE.
    vl_obj_name = 'ZPLC_PL'.
    v_pl        = 'PL'.
    v_plan_name = 'PL%9999999'.
  ENDIF.
***Inserção - Hermiro Carvalho - RDM 158441 - 23/02/2016 - Início
*  PERFORM gera_num_seq USING vl_obj_name CHANGING v_num_seq.
*  CLEAR: v_plan_name.
*  CONCATENATE v_pl v_num_seq INTO v_plan_name.
*  IF v_sobsl = c_30.
*    v_plan_name = 'BEXT%99999'.
*  ELSE.
*    v_plan_name = 'PL%9999999'.
*  ENDIF.
***Inserção - Hermiro Carvalho - RDM 158441 - 23/02/2016 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 01/02/2016 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 14/09/2015 - Fim
  FREE: t_bdc,
        t_return_bdc.

  PERFORM insere_bdc USING 'X' 'SAPMZRNPP_PLANO_CORTE'          '9000'.
  PERFORM insere_bdc USING ''  'ZRNPPT_PL_CORTE-NUM_PL_CORTE'   v_plan_name.
  PERFORM insere_bdc USING ''  'BDC_OKCODE'                     '=ENTE'.

  PERFORM insere_bdc USING 'X' 'SAPMZRNPP_PLANO_CORTE'          '9010'.
  PERFORM insere_bdc USING ''  'ZRNPPT_PL_CORTE-WERKS'          t_saida-werks.
***Inserção - Hermiro Carvalho - RDM 158714 - 17/03/2016 - Início
***Inserção - Hermiro Carvalho - RDM 158714 - 05/04/2016 - Início
  IF v_plan_name = 'BEXT%99999'.
    PERFORM insere_bdc USING ''  'ZRNPPT_PL_CORTE-LIFNR'          v_fornecedor.
  ENDIF.
***Inserção - Hermiro Carvalho - RDM 158714 - 05/04/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 158714 - 17/03/2016 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 24/09/2015 - Início
  IF t_vbap[] IS INITIAL.
    CLEAR: vl_string, vl_tabix, v_tabix_c, v_cont_aux.
***Inserção - Hermiro Carvalho - RDM 162929 - 20/05/2016 - Início
*    PERFORM insere_bdc USING 'X'   'SAPMZRNPP_PLANO_CORTE'          '9010'.
*    PERFORM insere_bdc USING ''    'BDC_SUBSCR'                     'SAPMZRNPP_PLANO_CORTE                   9050SUB_9010'.
*    PERFORM insere_bdc USING ''    'BDC_CURSOR'                     'S_MT9010-LOW'.
*    READ TABLE t_saida_ord INDEX 1.
*    PERFORM insere_bdc USING ''    'S_MT9010-LOW'                   t_saida_ord-matnr.
*    PERFORM insere_bdc USING 'X'   'BDC_CURSOR'                     'S_MT9010-LOW'.
    PERFORM insere_bdc USING ''    'BDC_OKCODE'                     '=%00190500000021552'.
***Inserção - Hermiro Carvalho - RDM 162929 - 20/05/2016 - Fim
    PERFORM insere_bdc USING 'X'   'SAPLALDB'                       '3000'.
    PERFORM insere_bdc USING ''    'BDC_OKCODE'                     '=ACPT'.
    PERFORM insere_bdc USING ''    'BDC_SUBSCR'                     'SAPLALDB                                3010SCREEN_HEADER'.
*    PERFORM insere_bdc USING ''    'BDC_CURSOR'        'RSCSEL_255-SLOW_I(01)'.

    CLEAR: t_matnr[].
    LOOP AT t_saida_ord.
      vl_tabix      = sy-tabix.
      v_cont_aux    = v_cont_aux + 1.
      v_tabix_c     = v_cont_aux.
      IF v_cont_aux <= 8."Número de linhas da tela de múltiplas seleção.
        CONCATENATE 'RSCSEL_255-SLOW_I(0' v_tabix_c ')' INTO vl_string.
        CONDENSE vl_string NO-GAPS.
        PERFORM insere_bdc USING ''   vl_string             t_saida_ord-matnr.
      ENDIF.
      IF v_cont_aux > 8."Adiciona o tab
        PERFORM insere_bdc USING ''    'BDC_OKCODE'                     '=P+'.
        v_cont_aux = 2.
        v_tabix_c = v_cont_aux.
        CONCATENATE 'RSCSEL_255-SLOW_I(0' v_tabix_c ')' INTO vl_string.
        CONDENSE vl_string NO-GAPS.
        PERFORM insere_bdc USING ''   vl_string             t_saida_ord-matnr.
      ENDIF.

      t_matnr-matnr = t_saida_ord-matnr.
      APPEND t_matnr.

    ENDLOOP.
  ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 24/09/2015 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 09/10/2015 - Início
  CLEAR: v_cont.
***Inserção - Hermiro Carvalho - PRJ01313 - 09/10/2015 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 16/10/2015 - Início
  CLEAR: t_ov[].
  LOOP AT t_vbap.

*    v_cont = v_cont + 1.
*
*    CLEAR v_numero.
*    WRITE v_cont TO v_numero.
*    CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
*      EXPORTING
*        input  = v_numero
*      IMPORTING
*        output = v_numero.
*
*    CLEAR: v_vbeln, v_posnr.
****Inserção - Hermiro Carvalho - PRJ01313 - 24/09/2015 - Início
*    CONCATENATE: 'T_OV-VBELN('
****Inserção - Hermiro Carvalho - PRJ01313 - 24/09/2015 - Fim
*                 v_numero ')'
*                 INTO v_vbeln.
*
*    CONCATENATE: 'T_OV-POSNR('
*                 v_numero ')'
*                 INTO v_posnr.
*
*    PERFORM insere_bdc USING ''  v_vbeln          t_vbap-vbeln.
*    PERFORM insere_bdc USING ''  v_posnr          t_vbap-posnr.
*    PERFORM insere_bdc USING ''  'BDC_OKCODE'     '/00'.
    t_ov-vbeln = t_vbap-vbeln.
    t_ov-posnr = t_vbap-posnr.
    APPEND t_ov.
  ENDLOOP.
  IF t_ov[] IS NOT INITIAL.
    SORT: t_ov BY vbeln posnr.
    DELETE ADJACENT DUPLICATES FROM t_ov COMPARING vbeln posnr.
    EXPORT t_ov TO MEMORY ID 'T_OVS_SEL'.
  ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 16/10/2015 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Início
  CASE sy-uname.
    WHEN 'RGC4691' OR 'RG01266'.
      wa_opt-dismode = 'A'.
      wa_opt-defsize = 8.
    WHEN OTHERS.
      wa_opt-dismode = 'E'.
      wa_opt-defsize = 8.
  ENDCASE.
***Inserção - Hermiro Carvalho - PRJ01313 - 13/10/2015 - Fim
*  CALL TRANSACTION 'ZPLCORTE' USING t_bdc MESSAGES INTO t_return_bdc OPTIONS FROM wa_opt.


  CLEAR: t_zplcorte[], t_zplcorte.
  t_zplcorte-num_pl_corte = v_plan_name.
  t_zplcorte-werks = t_saida-werks.
  t_zplcorte-lifnr = v_fornecedor.
  t_zplcorte-modelo = 'N'.
  t_zplcorte-separa_bobina = 'N'.
  APPEND t_zplcorte.

  SORT t_matnr BY matnr.
  DELETE ADJACENT DUPLICATES FROM t_matnr COMPARING matnr.

  EXPORT t_zplcorte TO MEMORY ID 'T_ZPLCORTE'.
  EXPORT t_matnr    TO MEMORY ID 'T_MATNR'.
  EXPORT t_ov       TO MEMORY ID 'T_OVS_SEL'.

  CALL TRANSACTION 'ZPLCORTE' AND SKIP FIRST SCREEN.

ENDFORM.                    " PLANO_DE_CORTE
*&---------------------------------------------------------------------*
*&      Form  GERA_NUM_SEQ
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      <--P_V_NUM_SEQ  text
*----------------------------------------------------------------------*
FORM gera_num_seq USING obj_name CHANGING num_seq.

  CALL FUNCTION 'NUMBER_GET_NEXT'
    EXPORTING
      nr_range_nr             = '01'
      object                  = obj_name
    IMPORTING
      number                  = num_seq
    EXCEPTIONS
      interval_not_found      = 1
      number_range_not_intern = 2
      object_not_found        = 3
      quantity_is_0           = 4
      quantity_is_not_1       = 5
      interval_overflow       = 6
      OTHERS                  = 7.


  CALL FUNCTION 'CONVERSION_EXIT_ALPHA_OUTPUT'
    EXPORTING
      input  = num_seq
    IMPORTING
      output = num_seq.

  CONDENSE num_seq.

ENDFORM.                    " GERA_NUM_SEQ
*&---------------------------------------------------------------------*
*&      Form  CONVERSION_VARIABLE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      <--P_WA_PLAF_KDAUF  text
*----------------------------------------------------------------------*
FORM conversion_variable  CHANGING variable.

  CALL FUNCTION 'CONVERSION_EXIT_ALPHA_OUTPUT'
    EXPORTING
      input  = variable
    IMPORTING
      output = variable.

ENDFORM.                    " CONVERSION_VARIABLE
*&---------------------------------------------------------------------*
*&      Module  STATUS_9040  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE status_9040 OUTPUT.
  SET PF-STATUS 'STATUS_9040'.
*  SET TITLEBAR 'xxx'.

ENDMODULE.                 " STATUS_9040  OUTPUT
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_9040  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_9040 INPUT.

  CASE sy-ucomm.
    WHEN 'CONF'.
      CLEAR: t_lotes_ovs[], t_lotes_ovs_aux[].
      LOOP AT t_mostra_lotes WHERE flag EQ 'X'.
        wa_lotes_prevp-charg = t_mostra_lotes-charg_mp.
***Inserção - Hermiro Carvalho - RDM 174798 - 15/02/2017 - Início
        wa_lotes_prevp-data_estoque = t_mostra_lotes-data_estoque.
***Inserção - Hermiro Carvalho - RDM 174798 - 15/02/2017 - Fim
        APPEND wa_lotes_prevp TO t_lotes_ovs.
        CLEAR: wa_lotes_prevp.
      ENDLOOP.
      EXPORT t_lotes_ovs TO MEMORY ID 'T_LOTES_9040'.

***Inserção - Hermiro Carvalho - RDM 174798 - 16/02/2016 - Início
      CLEAR: wa_lotes_prevp.
      LOOP AT t_mostra_lotes.
        wa_lotes_prevp-charg        = t_mostra_lotes-charg_mp.
        wa_lotes_prevp-data_estoque = t_mostra_lotes-data_estoque.
        APPEND wa_lotes_prevp TO t_lotes_ovs_aux.
        CLEAR: wa_lotes_prevp.
      ENDLOOP.
      EXPORT t_lotes_ovs_aux TO MEMORY ID 'T_LOTES_9040_AUX'.
***Inserção - Hermiro Carvalho - RDM 174798 - 16/02/2016 - Fim
      LEAVE TO SCREEN 0.

    WHEN 'CANCEL'.
      CLEAR: t_lotes_ovs[].
      LEAVE TO SCREEN 0.

    WHEN OTHERS.
  ENDCASE.

ENDMODULE.                 " USER_COMMAND_9040  INPUT
*&---------------------------------------------------------------------*
*&      Module  Z_LOTES_CHANGE_TC_ATTR  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE z_lotes_change_tc_attr OUTPUT.
  DESCRIBE TABLE t_mostra_lotes LINES z_lotes-lines.
ENDMODULE.                 " Z_LOTES_CHANGE_TC_ATTR  OUTPUT
*&---------------------------------------------------------------------*
*&      Module  Z_LOTES_MODIFY  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE z_lotes_modify INPUT.
  MODIFY t_mostra_lotes
    INDEX z_lotes-current_line.
ENDMODULE.                 " Z_LOTES_MODIFY  INPUT
*&---------------------------------------------------------------------*
*&      Form  COMPOSICOES
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM composicoes .

  DATA: v_werks     TYPE vbap-werks,
        v_kzkfg     TYPE mara-kzkfg,
        v_kzkfg_aux TYPE mara-kzkfg.

  DATA: t_saida_ord LIKE TABLE OF t_saida WITH HEADER LINE,
        t_pn_mrp    TYPE TABLE OF zqmt_partn_mrp WITH HEADER LINE,
        t_pn        TYPE TABLE OF zqmt_partnumber WITH HEADER LINE,
        lt_seltab   TYPE TABLE OF rsparams,
        ls_seltab   LIKE LINE OF lt_seltab,
        t_vbap_comp TYPE TABLE OF vbap WITH HEADER LINE,
        wa_mrp      TYPE ty_pn_mrp.

  DATA: t_stpo_comp TYPE TABLE OF ty_stpo,
        t_mast_comp TYPE TABLE OF ty_mast.

  READ TABLE t_saida WITH KEY mark = 'X'.

  IF sy-subrc <> 0.
    MESSAGE 'Nenhuma linha selecionada' TYPE 'E'.
  ENDIF.

  CLEAR: v_werks,
         v_kzkfg,
         v_kzkfg_aux,
         t_saida_ord,
         t_saida,
         t_mrp[].
  FREE: t_saida_ord.

  CLEAR: t_pn_mrp[], t_pn_mrp.
  CLEAR: t_pn[], t_pn.

  LOOP AT t_saida WHERE mark = 'X'.

    IF v_werks NE t_saida-werks AND v_werks IS NOT INITIAL.
      MESSAGE 'Não é permitido selecionar CENTROS distintos' TYPE 'E'.
    ENDIF.

    SELECT SINGLE kzkfg
      FROM mara
      INTO v_kzkfg
      WHERE matnr = t_saida-matnr.
***Inserção - Hermiro Carvalho - RDM 159510 - 14/03/2016 - Início
*    IF v_kzkfg NE v_kzkfg_aux AND v_kzkfg_aux IS NOT INITIAL.
*      MESSAGE 'Não é permitido selecionar Materiais com Tipo de Configuração distintas' TYPE 'E'.
*    ENDIF.
***Inserção - Hermiro Carvalho - RDM 159510 - 14/03/2016 - Fim
    MOVE-CORRESPONDING t_saida TO t_saida_ord.
    APPEND t_saida_ord.

    v_werks     = t_saida-werks.
    v_kzkfg_aux = v_kzkfg.

  ENDLOOP.


  IF v_kzkfg IS NOT INITIAL.

    "Cenário Configurável
    SELECT *
      INTO TABLE t_pn_mrp
      FROM zqmt_partn_mrp
      FOR ALL ENTRIES IN t_saida_ord
     WHERE werks  = t_saida_ord-werks
       AND idnrk  = t_saida_ord-materia_mp.
    CASE sy-subrc.
      WHEN 0."Existe
        SELECT *
          INTO TABLE t_pn
          FROM zqmt_partnumber
          FOR ALL ENTRIES IN t_pn_mrp
         WHERE partnumber =  t_pn_mrp-partnumber
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Início
           AND ( datefrom <= p_period-low  AND dateto >= p_period-high )
***Inserção - Hermiro Carvalho - PRJ01313 - 20/10/2015 - Fim
           AND atnam      = 'SD_MATERIAL_CLIENTE'
           AND ( status   = 'LIB' OR status = 'LIR' ).
    ENDCASE.

    LOOP AT t_pn_mrp.
      READ TABLE t_pn WITH KEY partnumber = t_pn_mrp-partnumber.
      IF sy-subrc IS INITIAL.
        wa_mrp-matnr      = t_pn-matnr.
        wa_mrp-partnumber = t_pn-kdmat.
        wa_mrp-werks      = t_pn_mrp-werks.
        APPEND wa_mrp TO t_mrp.
        CLEAR wa_mrp.
      ENDIF.
    ENDLOOP.

    IF t_mrp[] IS INITIAL.
      MESSAGE 'Não foi encontrada nenhum relação Partnumber x Matéria-Prima para composição' TYPE 'E'.
    ENDIF.

    CLEAR: t_vbap_comp[].
    SELECT a~vbeln a~posnr a~kdmat
        FROM vbap AS a INNER JOIN vbak AS b
        ON ( a~vbeln = b~vbeln )
        INTO CORRESPONDING FIELDS OF TABLE t_vbap_comp
        FOR ALL ENTRIES IN t_mrp
        WHERE a~matnr = t_mrp-matnr         AND
              a~kdmat = t_mrp-partnumber    AND
              a~werks = t_mrp-werks         AND
              b~auart  = '01'.
    IF t_vbap_comp[] IS INITIAL.
      MESSAGE 'Nenhuma ordem de venda de estoque(MD81) foi encontrada' TYPE 'E'.
    ENDIF.

    SORT t_vbap_comp BY vbeln posnr.
    DELETE ADJACENT DUPLICATES FROM t_vbap_comp COMPARING vbeln.

* verifica se existe planejada para as OVs de Demanda
    CLEAR: r_paart, r_paart[].
    r_paart-sign   = 'I'.
    r_paart-option = 'EQ'.
    r_paart-low = 'KD'. APPEND r_paart.
    r_paart-low = 'LA'. APPEND r_paart.
    CLEAR r_paart.

    CLEAR t_plaf_zovs_pendentes[].
    SELECT *
      FROM plaf
      INTO TABLE t_plaf_zovs_pendentes
      FOR ALL ENTRIES IN t_vbap_comp[]
      WHERE kdauf EQ t_vbap_comp-vbeln AND
            kdpos EQ t_vbap_comp-posnr.

    IF t_plaf_zovs_pendentes[] IS INITIAL.
      MESSAGE 'Nenhuma ordem planejada foi encontrada' TYPE 'E'.
    ELSE.
      DELETE ADJACENT DUPLICATES FROM t_vbap_comp COMPARING vbeln.
      CLEAR: lt_seltab[], ls_seltab.
      LOOP AT t_vbap_comp.
        ls_seltab-selname = 'S_VBELN'.
*      ls_seltab-kind    = 'S'.
        ls_seltab-sign    = 'I'.
        ls_seltab-option  = 'EQ'.
        ls_seltab-low     = t_vbap_comp-vbeln.
*      ls_seltab-high    = so_date-high.
        APPEND ls_seltab TO lt_seltab.
        CLEAR: ls_seltab.
      ENDLOOP.
      SUBMIT zsupp_ovprodext WITH p_data EQ ''
                             WITH p_vbeln EQ 'X'
                             WITH p_matnr EQ ''
                             WITH p_werks2 = v_werks
                             WITH SELECTION-TABLE lt_seltab
                             WITH p_b_e   EQ ''
                             WITH p_b_i   EQ ''
                             WITH p_tds   EQ 'X'
      AND RETURN.
    ENDIF.

  ELSE.

    "Cenário Configurado

    CLEAR: t_stpo_comp, t_mast_comp.

    SELECT stlty stlnr idnrk
      INTO TABLE t_stpo_comp
      FROM stpo
      FOR ALL ENTRIES IN t_saida_ord
     WHERE stlty =  'M'
       AND idnrk = t_saida_ord-materia_mp.
    CASE sy-subrc.
      WHEN 0.
        SELECT matnr werks stlan stlnr stlal
          INTO TABLE t_mast_comp
          FROM mast
          FOR ALL ENTRIES IN t_stpo_comp
         WHERE werks EQ v_werks
           AND stlan EQ '1'
           AND stlnr EQ t_stpo_comp-stlnr.
      WHEN OTHERS.
    ENDCASE.

    IF t_mast_comp[] IS INITIAL.
      MESSAGE s000(zehs) DISPLAY LIKE 'E' WITH 'Não foi encontrada nenhuma Lista Técnica para composição'.
      "      MESSAGE 'Não foi encontrada nenhuma Lista Técnica para composição' TYPE 'W'.
    ENDIF.

    SORT t_mast_comp BY matnr.
    DELETE ADJACENT DUPLICATES FROM t_mast_comp COMPARING matnr.

* verifica se existe planejada para os materiais
    CLEAR: r_paart, r_paart[].
    r_paart-sign   = 'I'.
    r_paart-option = 'EQ'.
    r_paart-low = 'KD'. APPEND r_paart.
    r_paart-low = 'LA'. APPEND r_paart.
    r_paart-low = 'NB'. APPEND r_paart.
    CLEAR r_paart.

    CLEAR t_plaf_zovs_pendentes[].
    SELECT *
      FROM plaf
      INTO TABLE t_plaf_zovs_pendentes
      FOR ALL ENTRIES IN t_mast_comp[]
      WHERE matnr EQ t_mast_comp-matnr AND
            pwwrk EQ t_mast_comp-werks AND
            paart IN r_paart.

    IF t_plaf_zovs_pendentes[] IS INITIAL.
      MESSAGE 'Nenhuma ordem planejada foi encontrada' TYPE 'E'.
    ELSE.
      CLEAR: lt_seltab[], ls_seltab.
      LOOP AT t_mast_comp INTO wa_mast.
        ls_seltab-selname = 'S_MATNR'.
*      ls_seltab-kind    = 'S'.
        ls_seltab-sign    = 'I'.
        ls_seltab-option  = 'EQ'.
        ls_seltab-low     = wa_mast-matnr.
*      ls_seltab-high    = so_date-high.
        APPEND ls_seltab TO lt_seltab.
        CLEAR: ls_seltab.
      ENDLOOP.
      SUBMIT zsupp_ovprodext WITH p_data EQ ''
                             WITH p_vbeln EQ ''
                             WITH p_matnr EQ 'X'
                             WITH p_werks3 = v_werks
                             WITH SELECTION-TABLE lt_seltab
                             WITH p_b_e   EQ ''
                             WITH p_b_i   EQ ''
                             WITH p_tds   EQ 'X'
      AND RETURN.
    ENDIF.

  ENDIF.

ENDFORM.                    " COMPOSICOES

*&---------------------------------------------------------------------*
*&      Form  F_FATURADO_PERIODO_REMESSA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM f_faturado_periodo_remessa .

  CONSTANTS c_lifsp  TYPE vbep-lifsp  VALUE 'Z1'.

  IF NOT t_saida[] IS INITIAL.

    CLEAR: it_vbak[], it_vbap[], it_vbep[], it_vbrp, it_vbep_vbap_vbak[], it_entped_it[].

    SELECT vbeln posnr etenr wmeng edatu lifsp
      INTO TABLE it_vbep
      FROM vbep
     WHERE edatu IN p_period
       AND lifsp NE c_lifsp.

    IF it_vbep[] IS NOT INITIAL.

      SELECT vbeln auart vkorg vtweg kunnr
        INTO TABLE it_vbak
        FROM vbak
        FOR ALL ENTRIES IN it_vbep
       WHERE vbeln  EQ it_vbep-vbeln
         AND auart  IN ('ZORB', 'ZOSV', 'ZXCS', 'ZSCS', 'ZSEM')
         AND augru   <> '14'
         AND augru   <> '64'
         AND kunnr  IN p_clt.
      SORT it_vbak BY vbeln.

      SELECT vbeln posnr werks matnr kdmat
       INTO TABLE it_vbap
       FROM vbap
        FOR ALL ENTRIES IN it_vbep
      WHERE vbeln  EQ it_vbep-vbeln
        AND posnr  EQ it_vbep-posnr
        AND matnr  IN p_mat.
***Inserção - Sidney Q de Andrade - INC 2170534 - 20/07/2016 - Inicio
*        AND werks  IN p_centro.
***Inserção - Sidney Q de Andrade - INc 2170534 - 20/07/2016 - Fim
      SORT: it_vbep BY vbeln posnr.

    ENDIF.

    LOOP AT it_vbak INTO wa_vbak_mapa.
      LOOP AT it_vbap INTO wa_vbap_mapa WHERE vbeln = wa_vbak_mapa-vbeln.
        LOOP AT it_vbep INTO wa_vbep_mapa WHERE vbeln = wa_vbap_mapa-vbeln
                                       AND posnr = wa_vbap_mapa-posnr.
          it_vbep_vbap_vbak-werks = wa_vbap_mapa-werks.
          it_vbep_vbap_vbak-matnr = wa_vbap_mapa-matnr.
          it_vbep_vbap_vbak-vbeln = wa_vbep_mapa-vbeln.
          it_vbep_vbap_vbak-posnr = wa_vbep_mapa-posnr.
          it_vbep_vbap_vbak-kdmat = wa_vbap_mapa-kdmat.
          it_vbep_vbap_vbak-etenr = wa_vbep_mapa-etenr.
          it_vbep_vbap_vbak-kunnr = wa_vbak_mapa-kunnr.
          APPEND it_vbep_vbap_vbak.
        ENDLOOP.
      ENDLOOP.
    ENDLOOP.

    IF it_vbep_vbap_vbak[] IS NOT INITIAL.

      SELECT *
        INTO TABLE it_entped_it
        FROM zsuppt_entped_it
        FOR ALL ENTRIES IN it_vbep_vbap_vbak
        WHERE werks = it_vbep_vbap_vbak-werks
          AND vbeln = it_vbep_vbap_vbak-vbeln
          AND posnr = it_vbep_vbap_vbak-posnr
          AND etenr = it_vbep_vbap_vbak-etenr.

      IF sy-subrc IS INITIAL.
        SORT it_entped_it BY werks vbeln posnr etenr.
      ENDIF.

    ENDIF.

    IF it_entped_it[] IS NOT INITIAL.

      SELECT p~vbeln p~posnr p~vgbel p~aubel p~aupos p~matnr p~ntgew p~fkimg p~werks
             p~erdat k~xblnr
        INTO TABLE it_vbrp
        FROM vbrp AS p INNER JOIN vbrk AS k
          ON p~vbeln = k~vbeln
        FOR ALL ENTRIES IN it_entped_it
         WHERE p~vgbel = it_entped_it-vgbel
           AND p~aubel = it_entped_it-vbeln
           AND p~aupos = it_entped_it-posnr
***Inserção - Hermiro Carvalho - RDM 181010 - 29/05/2017 - Início
           AND p~erdat NOT IN p_period.
***Inserção - Hermiro Carvalho - RDM 181010 - 29/05/2017 - Fim

      SORT it_vbrp BY vgbel aubel aupos.

    ENDIF.

    LOOP AT t_saida.

      v_tabix = sy-tabix.

      LOOP AT it_vbep_vbap_vbak WHERE matnr = t_saida-matnr
                                  AND kdmat = t_saida-codigo
                                  AND kunnr = t_saida-cod_cli.

        LOOP AT it_entped_it WHERE werks = it_vbep_vbap_vbak-werks AND
                                   vbeln = it_vbep_vbap_vbak-vbeln AND
                                   posnr = it_vbep_vbap_vbak-posnr AND
                                   etenr = it_vbep_vbap_vbak-etenr.

***Inserção - Hermiro Carvalho - RDM 181010 - 29/05/2017 - Início
          READ TABLE it_vbrp WITH KEY vgbel = it_entped_it-vgbel
                                      aubel = it_entped_it-vbeln
                                      aupos = it_entped_it-posnr.
          IF sy-subrc IS NOT INITIAL.
            CONTINUE.
          ENDIF.
***Inserção - Hermiro Carvalho - RDM 181010 - 29/05/2017 - Fim
          IF it_vbep_vbap_vbak-werks = t_saida-werks.
            t_saida-faturado    = t_saida-faturado + it_entped_it-peso_liq.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*            t_saida-faturado_pc = t_saida-faturado_pc + it_entped_it-fkimg.
            IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
              v_pecas = it_entped_it-fkimg.
            ELSE.
              IF t_saida-peso_peca IS NOT INITIAL.
                v_pecas = it_entped_it-fkimg / ( t_saida-peso_peca / 1000 ).
              ENDIF.
            ENDIF.
            t_saida-faturado_pc = t_saida-faturado_pc + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim
          ELSE.
            t_saida-faturado_cen    = t_saida-faturado_cen + it_entped_it-peso_liq.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Inicio
*            t_saida-faturado_cen_pc = t_saida-faturado_cen_pc + it_entped_it-fkimg.
            IF t_saida-meins = 'ST' OR t_saida-meins = 'PC'.
              v_pecas = it_entped_it-fkimg.
            ELSE.
              IF t_saida-peso_peca IS NOT INITIAL.
                v_pecas = it_entped_it-fkimg / ( t_saida-peso_peca / 1000 ).
              ENDIF.
            ENDIF.
            t_saida-faturado_cen_pc = t_saida-faturado_cen_pc + v_pecas.
***Inserção - Sidney Quintão de Andrade - INC 2183836 - Fim
          ENDIF.

        ENDLOOP.

      ENDLOOP.

      MODIFY t_saida INDEX v_tabix.

      CLEAR: t_saida, v_pecas.

    ENDLOOP.

  ENDIF.

ENDFORM.                    " F_FATURADO_PERIODO_REMESSA

*&---------------------------------------------------------------------*
*&      Form  F_CARREGA_BEXT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM f_carrega_bext .

  DATA: lc_entrada TYPE ekbe-menge,
        vl_strgr   TYPE marc-strgr.
* REQUISICOES DE COMPRA
  CLEAR: t_eban[].

  IF t_melhoria[] IS NOT INITIAL.

    SELECT banfn bnfpo matnr
           werks ebeln ebelp
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Início
           menge ebakz zzvbeln zzposnr meins
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Fim
    FROM eban
     INTO TABLE t_eban
***Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Início
*   FOR ALL ENTRIES IN t_saida
     FOR ALL ENTRIES IN t_melhoria
     WHERE bsart = 'NB'
       AND bstyp = 'B'
       AND loekz = ' '
       AND pstyp = '3'
*     AND matnr EQ t_saida-matnr
*     AND werks EQ t_saida-wrk02
       AND matnr EQ t_melhoria-matnr
       AND werks EQ t_melhoria-wrk02 " 09mar2017 RGC8096 RDM: 176853
*      AND werks EQ t_melhoria-werks
***Inserção - Hermiro Carvalho - RDM 160090 - 04/05/2016 - Fim
*Inserção - RDM 159206 - Inicio
*     AND erdat IN p_period
*Inserção - RDM 159206 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 06/11/2015 - Início
*     AND banfn EQ ''.
        AND ebeln EQ ''
***Inserção - Sidney Q de Andrade - RDM 167106 - 11/08/2016 - Início
        AND ebakz EQ ''. "Status Concluída
***Inserção - Sidney Q de Andrade - RDM 167106 - 11/08/2016 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 06/11/2015 - Fim
*  DELETE t_eban WHERE ebakz = 'X'.    "Descartar Requisições Concluidas
***Comentar - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Início
*  DELETE t_eban WHERE zzvbeln IS INITIAL. "Descartar Requisições sem OVs
***Comentar - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Fim

  ENDIF.
* >>> - 06.08.2019 15:26:22 - RDM 223715 - Simone
  IF t_eban[] IS NOT INITIAL.
    DELETE t_eban WHERE matnr(2) EQ '12'.
  ENDIF.
* <<< - 06.08.2019 15:26:22 - RDM 223715 - Simone
  IF t_eban[] IS NOT INITIAL.

    CLEAR: v_tabix.

    LOOP AT t_saida.

      v_tabix = sy-tabix.

      CLEAR: t_saida-requisicoes, t_saida-requisicoes_pc.

      LOOP AT t_eban INTO wa_eban WHERE matnr = t_saida-matnr
                                    AND werks = t_saida-wrk02.
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Início
*        CLEAR v_kunnr.
*        SELECT kunnr UP TO 1 ROWS
*          INTO v_kunnr
*          FROM vbak
*          WHERE vbeln = wa_eban-zzvbeln.
*        ENDSELECT.
*        IF v_kunnr IS INITIAL.
*
*          CLEAR v_pni.
*          SELECT SINGLE partnumber
*            FROM zqmt_partn_mrp
*            INTO v_pni
*           WHERE vbeln = wa_eban-zzvbeln.
*
*          SELECT SINGLE kunnr
*            FROM zqmt_partnumber
*            INTO v_kunnr
*           WHERE partnumber = v_pni AND
*                 atnam      = 'SD_MATERIAL_CLIENTE' AND
*               ( status     = 'LIB' OR status = 'LIR' ).
*        ENDIF.
*
*        CLEAR v_kdmat.
*        SELECT kdmat UP TO 1 ROWS
*          INTO v_kdmat
*          FROM vbap
*          WHERE vbeln = wa_eban-zzvbeln
*            AND posnr = wa_eban-zzposnr.
*        ENDSELECT.
        PERFORM f_verifica_estrategia USING t_saida-matnr
                                            t_saida-wrk02
                                   CHANGING vl_strgr.
        IF vl_strgr EQ '25'.

          CLEAR v_kdmat.
          SELECT SINGLE kdmat
            FROM vbap
            INTO v_kdmat
            WHERE vbeln = wa_eban-zzvbeln AND
                  posnr = wa_eban-zzposnr.

          CLEAR: v_kunnr.
          SELECT SINGLE kunnr
            INTO v_kunnr
            FROM vbak
           WHERE vbeln = wa_eban-zzvbeln.

          CLEAR t_zrnppt_prev.
          READ TABLE t_zrnppt_prev WITH KEY werks  = t_saida-wrk02
                                            matnr  = t_saida-matnr
                                            codigo = v_kdmat
                                            kunnr  = v_kunnr.
          IF sy-subrc EQ 0.
            CLEAR: v_kunnr, v_kdmat.
          ENDIF.

        ELSE.

          CLEAR t_zrnppt_prev.
          READ TABLE t_zrnppt_prev WITH KEY werks = t_saida-wrk02
                                            matnr = t_saida-matnr.
          IF sy-subrc EQ 0.
            v_kunnr = t_zrnppt_prev-kunnr.
            v_kdmat = t_zrnppt_prev-codigo.
          ENDIF.

        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Fim
        IF t_saida-cod_cli <> v_kunnr.
          CONTINUE.
        ENDIF.

        IF t_saida-codigo <> v_kdmat.
          CONTINUE.
        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Início
        IF wa_eban-meins = 'PC' OR wa_eban-meins = 'ST'.
          IF ( t_saida-peso_peca > 0 ).
            wa_eban-menge = wa_eban-menge * ( t_saida-peso_peca / 1000 ).
          ELSE.
            CLEAR wa_eban-menge.
          ENDIF.
        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Fim
        t_saida-requisicoes = t_saida-requisicoes + wa_eban-menge.

      ENDLOOP.

      IF ( t_saida-peso_peca > 0 ).
        t_saida-requisicoes_pc = ( t_saida-requisicoes * 1000 ) / t_saida-peso_peca.
      ENDIF.

      MODIFY t_saida INDEX v_tabix.

    ENDLOOP.

  ENDIF.

* PEDIDOS DE COMPRA
  CLEAR t_ekko[].

  IF t_melhoria[] IS NOT INITIAL.

    SELECT a~ebeln a~bukrs a~ernam a~lifnr a~bedat
           b~ebelp b~matnr b~werks b~menge b~meins b~netpr
           b~brtwr b~anfnr b~anfps b~banfn b~bnfpo
      INTO TABLE t_ekko
      FROM ekko AS a INNER JOIN ekpo AS b
        ON a~ebeln = b~ebeln
        FOR ALL ENTRIES IN t_melhoria
*      FOR ALL ENTRIES IN t_saida
*Inserção - RDM 159206 – Inicio
*      WHERE a~bedat IN p_period AND
*              matnr EQ t_saida-matnr AND
*              werks EQ t_saida-wrk02.
        WHERE b~matnr EQ t_melhoria-matnr AND
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
*              b~werks EQ t_melhoria-wrk02 AND "09Mmar2017 RGC8096 RDM 176853
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
*               werks EQ t_melhoria-werks AND
              b~elikz NE 'X'              AND "Remessa final <> Não
              b~loekz NE 'L'              AND "09mar2017 RGC8096 RDM: 176853
              b~pstyp EQ '3'              AND "Subcontratação
              a~bsart EQ 'NB'.                "Tipo de Documento
*Inserção - RDM 159206 - Fim

  ENDIF.

* >>> - 06.08.2019 15:26:22 - RDM 223715 - Simone
  IF t_ekko[] IS NOT INITIAL.
    DELETE t_ekko WHERE matnr(2) EQ '12'.
  ENDIF.
* <<< - 06.08.2019 15:26:22 - RDM 223715 - Simone

  IF t_ekko[] IS NOT INITIAL.

*    PERFORM atual_tabaux.

*INC 3258470
*    SELECT banfn bnfpo matnr
*           werks ebeln ebelp
*           menge ebakz zzvbeln zzposnr
*        FROM eban
*        INTO TABLE t_eban_aux
*          FOR ALL ENTRIES IN t_ekko
*        WHERE ebeln = t_ekko-ebeln
*          AND ebelp = t_ekko-ebelp.
*INC 3258470

    SELECT ebeln ebelp belnr buzei menge cpudt cputm matnr charg
      FROM ekbe
      INTO TABLE t_ekbe
      FOR ALL ENTRIES IN t_ekko
      WHERE ebeln = t_ekko-ebeln
        AND ebelp = t_ekko-ebelp
        AND ( bwart = '101' OR bwart = '105' ).

***Inserção - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Início
    SORT t_ekko BY ebeln ASCENDING
                   ebelp ASCENDING
                   werks ASCENDING
                   matnr ASCENDING
                   lifnr ASCENDING.
*
*    SORT t_ekko_aux BY ebeln ASCENDING
*                       ebelp ASCENDING
*                       werks ASCENDING
*                       matnr ASCENDING
*                       lifnr ASCENDING.
*
*    SORT t_eban BY banfn ASCENDING
*                   bnfpo ASCENDING.

*INC 3258470
*    SORT t_eban_aux BY ebeln ASCENDING
*                       ebelp ASCENDING.
*INC 3258470

***Inserção - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Fim

    SORT: t_ekbe BY ebeln ASCENDING
                    ebelp ASCENDING.

    CLEAR: v_tabix.

    LOOP AT t_saida.

      v_tabix = sy-tabix.

      CLEAR: t_saida-pedidos, t_saida-pedidos_pc.

      LOOP AT t_ekko INTO wa_ekko WHERE matnr = t_saida-matnr.
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
*                                    AND werks = t_saida-wrk02.
*PP - Criar coluna POs Outros Centros na GD - RDM 244981

***Inserção - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Início
*        CLEAR wa_ekko_aux.
*        READ TABLE t_ekko_aux INTO wa_ekko_aux WITH KEY ebeln(10) = wa_ekko-ebeln
*                                                        ebelp     = wa_ekko-ebelp
*                                                        BINARY SEARCH.

*INC 3258470
*        CLEAR: wa_eban.
*        READ TABLE t_eban_aux INTO wa_eban WITH KEY ebeln = wa_ekko-ebeln
*                                                    ebelp = wa_ekko-ebelp
*                                                    BINARY SEARCH.
*INC 3258470

*        IF sy-subrc = 0.

*        CLEAR wa_eban.
*        READ TABLE t_eban INTO wa_eban WITH KEY banfn = wa_ekko_aux-banfn
*                                                bnfpo = wa_ekko_aux-bnfpo
*                                                BINARY SEARCH.
***Inserção - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Início
*        CLEAR v_kunnr.
*        SELECT kunnr UP TO 1 ROWS
*          INTO v_kunnr
*          FROM vbak
*          WHERE vbeln = wa_eban-zzvbeln.
*        ENDSELECT.
*
*        IF v_kunnr IS INITIAL.
*          CLEAR v_pni.
*          SELECT SINGLE partnumber
*            FROM zqmt_partn_mrp
*            INTO v_pni
*           WHERE vbeln = wa_eban-zzvbeln.
*
*          SELECT SINGLE kunnr
*            FROM zqmt_partnumber
*            INTO v_kunnr
*           WHERE partnumber = v_pni AND
*                 atnam      = 'SD_MATERIAL_CLIENTE' AND
*               ( status     = 'LIB' OR status = 'LIR' ).
*        ENDIF.
*
*        CLEAR v_kdmat.
*        SELECT kdmat UP TO 1 ROWS
*          INTO v_kdmat
*          FROM vbap
*          WHERE vbeln = wa_eban-zzvbeln
*            AND posnr = wa_eban-zzposnr.
*        ENDSELECT.

*INC 3258470
*        PERFORM f_verifica_estrategia USING t_saida-matnr
*                                            t_saida-werks
*                                      CHANGING vl_strgr.
*        IF vl_strgr EQ '25'.
*
*          CLEAR v_kdmat.
*          SELECT SINGLE kdmat
*            FROM vbap
*            INTO v_kdmat
*            WHERE vbeln = wa_eban-zzvbeln AND
*                  posnr = wa_eban-zzposnr.
*
*          CLEAR: v_kunnr.
*          SELECT SINGLE kunnr
*            INTO v_kunnr
*            FROM vbak
*           WHERE vbeln = wa_eban-zzvbeln.
*
*          CLEAR t_zrnppt_prev.
*          READ TABLE t_zrnppt_prev WITH KEY werks  = t_saida-werks
*                                            matnr  = t_saida-matnr
*                                            codigo = v_kdmat
*                                            kunnr  = v_kunnr.
*          IF sy-subrc EQ 0.
*            CLEAR: v_kunnr, v_kdmat.
*          ENDIF.
*
*        ELSE.
*INC 3258470

        CLEAR t_zrnppt_prev.
        READ TABLE t_zrnppt_prev WITH KEY werks = t_saida-werks "rk02 10mar2017 RGC8096
                                          matnr = t_saida-matnr.
        IF sy-subrc EQ 0.
          v_kunnr = t_zrnppt_prev-kunnr.
          v_kdmat = t_zrnppt_prev-codigo.
        ENDIF.

*INC 3258470
*        ENDIF.
*INC 3258470

***Inserção - Hermiro Carvalho - PRJ01313 - 02/02/2016 - Fim
        IF t_saida-cod_cli <> v_kunnr.
          CONTINUE.
        ENDIF.

        IF t_saida-codigo <> v_kdmat.
          CONTINUE.
        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Início
        IF wa_ekko-meins = 'PC' OR wa_ekko-meins = 'ST'.
          IF ( t_saida-peso_peca > 0 ).
            wa_ekko-menge = wa_ekko-menge * ( t_saida-peso_peca / 1000 ).
          ELSE.
            CLEAR wa_ekko-menge.
          ENDIF.
        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Fim
        CLEAR lc_entrada.
        LOOP AT t_ekbe WHERE ebeln = wa_ekko-ebeln AND
                             ebelp = wa_ekko-ebelp.
***Inserção - Hermiro Carvalho - INC 2410544 - 25/08/2017 - Início
          CLEAR: wa_mseg_log, v_answer.
          PERFORM verifica_tipo_movimento USING t_ekbe-matnr
                                                t_ekbe-charg
                                                t_ekbe-belnr
                                                t_ekbe-buzei
                                       CHANGING v_answer.
          IF v_answer IS NOT INITIAL.
            CONTINUE.
          ENDIF.
***Inserção - Hermiro Carvalho - INC 2410544 - 25/08/2017 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Início
          IF wa_ekko-meins = 'PC' OR wa_ekko-meins = 'ST'.
            IF ( t_saida-peso_peca > 0 ).
              t_ekbe-menge = t_ekbe-menge * ( t_saida-peso_peca / 1000 ).
            ELSE.
              CLEAR t_ekbe-menge.
            ENDIF.
          ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 29/01/2016 - Fim
          lc_entrada = t_ekbe-menge + lc_entrada.
        ENDLOOP.

**PP - Criar coluna POs Outros Centros na GD - RDM 244981
        IF wa_ekko-werks NE t_saida-werks.
          t_saida-pedidos_cen = t_saida-pedidos_cen + wa_ekko-menge.
          t_saida-pedidos_cen = t_saida-pedidos_cen - lc_entrada.
          IF t_saida-pedidos_cen < 0.
            CLEAR t_saida-pedidos_cen.
          ENDIF.
        ELSE.
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
          t_saida-pedidos = t_saida-pedidos + wa_ekko-menge.
          t_saida-pedidos = t_saida-pedidos - lc_entrada.
          IF t_saida-pedidos < 0.
            CLEAR t_saida-pedidos.
          ENDIF.
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
        ENDIF.
*PP - Criar coluna POs Outros Centros na GD - RDM 244981

***Inserção - Hermiro Carvalho - RDM 184185 - 19/09/2017 - Início
        IF t_saida-fornecedor IS INITIAL.
          SELECT SINGLE sortl
            INTO t_saida-fornecedor
            FROM lfa1
           WHERE lifnr =  wa_ekko-lifnr.
*          t_saida-fornecedor = wa_ekko-lifnr.
        ELSE.
          IF t_saida-fornecedor <> wa_ekko-lifnr.
            t_saida-fornecedor = '*'.
          ENDIF.
        ENDIF.
***Inserção - Hermiro Carvalho - RDM 184185 - 19/09/2017 - Fim
***Inserção - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Início
*        ENDIF.
***Inserção - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Fim
      ENDLOOP.

      IF ( t_saida-peso_peca > 0 ).
        t_saida-pedidos_pc = ( t_saida-pedidos * 1000 ) / t_saida-peso_peca.

*PP - Criar coluna POs Outros Centros na GD - RDM 244981
        t_saida-pedidos_cen_pc = ( t_saida-pedidos_cen * 1000 ) / t_saida-peso_peca.
*PP - Criar coluna POs Outros Centros na GD - RDM 244981
      ENDIF.

      MODIFY t_saida INDEX v_tabix.

    ENDLOOP.

  ENDIF.

ENDFORM.                    " F_CARREGA_BEXT

*&---------------------------------------------------------------------*
*&      Form  ATUAL_TABAUX
*&---------------------------------------------------------------------*
*&      Atualiza tabela auxiliar
*&---------------------------------------------------------------------*
FORM atual_tabaux .
***Comentar - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Início
*  LOOP AT t_ekko INTO wa_ekko.
*
*    wa_ekko_aux-ebeln = wa_ekko-ebeln.
*    wa_ekko_aux-bukrs = wa_ekko-bukrs.
*    wa_ekko_aux-ernam = wa_ekko-ernam.
*    wa_ekko_aux-lifnr = wa_ekko-lifnr.
*    wa_ekko_aux-bedat = wa_ekko-bedat.
*    wa_ekko_aux-ebelp = wa_ekko-ebelp.
*    wa_ekko_aux-matnr = wa_ekko-matnr.
*    wa_ekko_aux-werks = wa_ekko-werks.
*    wa_ekko_aux-menge = wa_ekko-menge.
*    wa_ekko_aux-meins = wa_ekko-meins.
*    wa_ekko_aux-netpr = wa_ekko-netpr.
*    wa_ekko_aux-anfnr = wa_ekko-anfnr.
*    wa_ekko_aux-banfn = wa_ekko-banfn.
*    wa_ekko_aux-bnfpo = wa_ekko-bnfpo.
*
*    APPEND wa_ekko_aux TO t_ekko_aux.
*    CLEAR: wa_ekko_aux, wa_ekko.
*
*  ENDLOOP.
***Comentar - Hermiro Carvalho - PRJ01313 - 12/11/2015 - Fim
ENDFORM.                    " ATUAL_TABAUX
*&---------------------------------------------------------------------*
*&      Form  Z_SEPARA_SELECIONADOS
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      <--P_V_SOBSL  text
*----------------------------------------------------------------------*
FORM z_separa_selecionados  CHANGING p_v_sobsl TYPE sobsl.

  DATA lc_sobsl TYPE marc-sobsl.

  CLEAR v_erro.
*  REFRESH t_processar.

  CLEAR v_sobsl.

  LOOP AT t_saida.

    IF t_saida-mark EQ 'X'.
      SELECT SINGLE sobsl
        FROM marc
        INTO lc_sobsl
        WHERE matnr EQ t_saida-matnr
          AND werks EQ t_saida-werks.

      IF v_sobsl NE lc_sobsl AND v_sobsl IS NOT INITIAL.
        MESSAGE 'Não é possível selecionar mais de um registro com Suprimentos diferentes' TYPE 'E'.
      ENDIF.

      v_sobsl = lc_sobsl.

    ENDIF.

  ENDLOOP.

ENDFORM.                    " Z_SEPARA_SELECIONADOS
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_9150  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_9150 INPUT.

  DATA: vl_lifnr TYPE lfa1-lifnr.
  CLEAR: vl_lifnr.
  CASE sy-ucomm.
    WHEN 'BTNMP'.
      IF v_fornecedor IS NOT INITIAL.
***Inserção - Hermiro Carvalho - RDM 158714 - 10/03/2016 - Início
        CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
          EXPORTING
            input  = v_fornecedor
          IMPORTING
            output = v_fornecedor.
***Inserção - Hermiro Carvalho - RDM 158714 - 10/03/2016 - Fim
        SELECT SINGLE lifnr
          INTO vl_lifnr
          FROM lfa1
         WHERE lifnr = v_fornecedor.
        CASE sy-subrc.
          WHEN 0.
            v_resp = 1.
            LEAVE TO SCREEN 0.
          WHEN OTHERS.
            MESSAGE 'Fornecedor inexistente no cadastro' TYPE 'I'.
            MESSAGE 'Fornecedor inexistente no cadastro' TYPE 'S' DISPLAY LIKE 'E'.
            RETURN.
        ENDCASE.
      ENDIF.
    WHEN 'BTNEST'.
      IF v_fornecedor IS NOT INITIAL.
***Inserção - Hermiro Carvalho - RDM 158714 - 10/03/2016 - Início
        CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
          EXPORTING
            input  = v_fornecedor
          IMPORTING
            output = v_fornecedor.
***Inserção - Hermiro Carvalho - RDM 158714 - 10/03/2016 - Fim
        SELECT SINGLE lifnr
          INTO vl_lifnr
          FROM lfa1
         WHERE lifnr = v_fornecedor.
        CASE sy-subrc.
          WHEN 0.
            v_resp = 1.
            LEAVE TO SCREEN 0.
          WHEN OTHERS.
            MESSAGE 'Fornecedor inexistente no cadastro' TYPE 'I'.
            MESSAGE 'Fornecedor inexistente no cadastro' TYPE 'S' DISPLAY LIKE 'E'.
            RETURN.
        ENDCASE.
      ENDIF.
    WHEN 'BTNCANC'.
      v_resp = 3.
      LEAVE TO SCREEN 0.
    WHEN OTHERS.
  ENDCASE.

ENDMODULE.                 " USER_COMMAND_9150  INPUT
*&---------------------------------------------------------------------*
*&      Module  STATUS_9150  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE status_9150 OUTPUT.
  SET PF-STATUS 'STATUS_9150'.
  SET TITLEBAR 'TITLE_9150'.

ENDMODULE.                 " STATUS_9150  OUTPUT

*&---------------------------------------------------------------------*
*&      Form  CHECK_RESERVA_ESTOQUE_ESPECIAL
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_WA_MSLB_MATNR  text
*      -->P_WA_MSLB_WERKS  text
*      -->P_WA_MSLB_CHARG  text
*      <--P_MATCHES  text
*----------------------------------------------------------------------*
FORM check_reserva_estoque_especial  USING    p_matnr
                                              p_werks
                                              p_charg
                                     CHANGING p_log_mslb.

  DATA: t_calc_comp  TYPE TABLE OF zsummt_calc_comp,
        t_pl_itens   TYPE TABLE OF zrnppt_pl_itens,
        t_ekpo       TYPE TABLE OF ekpo,
        t_resb       TYPE TABLE OF resb,
        wa_ekpo      TYPE ekpo,
        wa_resb      TYPE resb,
        wa_pl_itens  TYPE zrnppt_pl_itens,
        wa_calc_comp TYPE zsummt_calc_comp.

  CLEAR: t_calc_comp[], t_pl_itens[], t_ekpo[], t_resb[], wa_ekpo, wa_resb, wa_pl_itens, wa_calc_comp.

  SELECT centro num_pl_corte it_pp vbeln posnr lote matnr lgort
    INTO CORRESPONDING FIELDS OF TABLE t_calc_comp
    FROM zsummt_calc_comp
   WHERE centro = p_werks
     AND lote   = p_charg
     AND matnr  = p_matnr.
  CASE sy-subrc."Verifica se encontrou dados na tabela zsummt_calc_comp - Início
    WHEN 0."Encontrou: zsummt_calc_comp
      SELECT werks num_pl_corte arbpl seq_atual item ebeln ebelp
        INTO CORRESPONDING FIELDS OF TABLE t_pl_itens
        FROM zrnppt_pl_itens
        FOR ALL ENTRIES IN t_calc_comp
       WHERE werks         = t_calc_comp-centro
         AND num_pl_corte  = t_calc_comp-num_pl_corte.
      SORT t_pl_itens BY num_pl_corte.
      DELETE ADJACENT DUPLICATES FROM t_pl_itens COMPARING num_pl_corte.
***Inserção - Hermiro Carvalho - INC 2410544 - 25/08/2017 - Início
*      CASE sy-subrc."Verifica se encontrou dados na tabela zrnppt_pl_itens - Início
*        WHEN 0."Encontrou: zrnppt_pl_itens
      IF t_pl_itens[] IS NOT INITIAL."Verifica se encontrou dados na tabela zrnppt_pl_itens - Início
***Inserção - Hermiro Carvalho - INC 2410544 - 25/08/2017 - Fim
        LOOP AT t_pl_itens INTO wa_pl_itens.
          IF wa_pl_itens-ebeln IS NOT INITIAL AND wa_pl_itens-ebelp IS NOT INITIAL."Verifica se estão preenchidos - Início
            SELECT SINGLE ebeln ebelp matnr werks elikz
              INTO CORRESPONDING FIELDS OF wa_ekpo
              FROM ekpo
             WHERE ebeln  = wa_pl_itens-ebeln
               AND ebelp  = wa_pl_itens-ebelp.
            CASE sy-subrc."Verifica se encontrou registro na tabela EKPO - Início
              WHEN 0.
                IF wa_ekpo-elikz EQ space.
                  CONCATENATE 'Lote alocado no Pedido de Compra' wa_pl_itens-ebeln INTO p_log_mslb SEPARATED BY space.
                  EXIT.
                ENDIF.
              WHEN OTHERS.
                CLEAR: wa_pl_itens.
                CONTINUE.
            ENDCASE."Verifica se encontrou registro na tabela EKPO - Fim
          ENDIF."Verifica se estão preenchidos - Fim

          IF wa_pl_itens-ebeln IS INITIAL AND wa_pl_itens-ebelp IS INITIAL."Verifica se não estão preenchidos - Início
            SELECT rsnum rspos rsart ebeln ebelp
              INTO CORRESPONDING FIELDS OF TABLE t_resb
              FROM resb
              FOR ALL ENTRIES IN t_calc_comp
             WHERE matnr = t_calc_comp-matnr
               AND werks = t_calc_comp-centro
               AND charg = t_calc_comp-lote.
**                 and ebeln <>
            CASE sy-subrc."Verifica se encontrou registro na RESB - Início
              WHEN 0."Encontrou: Resb
                LOOP AT t_resb INTO wa_resb.
***Inserção - Hermiro Carvalho - INC 2128326 - 05/05/2016 - Início
*                    SELECT SINGLE ebeln ebelp matnr werks elikz
                  SELECT SINGLE *
***Inserção - Hermiro Carvalho - INC 2128326 - 05/05/2016 - Fim
                    INTO wa_ekpo
                    FROM ekpo
                   WHERE ebeln = wa_resb-ebeln
                     AND ebelp = wa_resb-ebelp.
                  CASE sy-subrc."Verifica se encontrou registro na tabela EKPO - Início
                    WHEN 0.
                      IF wa_ekpo-elikz EQ space.
                        CONCATENATE 'Lote alocado no Pedido de Compra' wa_resb-ebeln INTO p_log_mslb SEPARATED BY space.
                        EXIT.
                      ENDIF.
                    WHEN OTHERS.
                      CLEAR: wa_resb.
                      CONTINUE.
                  ENDCASE.
                ENDLOOP.
              WHEN OTHERS."Não encontrou: Resb
            ENDCASE."Verifica se encontrou registro na RESB - Fim
          ENDIF."Verifica se não estão preenchidos - Fim

        ENDLOOP.
***Inserção - Hermiro Carvalho - INC 2410544 - 25/08/2017 - Início
*        WHEN OTHERS."Não Encontrou: zrnppt_pl_itens
*      ENDCASE."Verifica se encontrou dados na tabela zrnppt_pl_itens - Fim
      ENDIF.
***Inserção - Hermiro Carvalho - INC 2410544 - 25/08/2017 - Fim
    WHEN OTHERS."Não Encontrou: zsummt_calc_comp
  ENDCASE."Verifica se encontrou dados na tabela zsummt_calc_comp - Fim

ENDFORM.                    " CHECK_RESERVA_ESTOQUE_ESPECIAL
*&---------------------------------------------------------------------*
*&      Form  APURA_SALDO_MP
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM apura_saldo_mp .
  v_wmeng = t_saida-aproduzir.

  CLEAR: v_neces_mp, v_neces_mp_aux.

  v_neces_mp_aux =  v_wmeng.
  v_neces_mp     =  v_neces_mp + v_neces_mp_aux.

* Se total a produzir for negativo, o campo necessidade MP fica igual á zero.
  IF t_saida-aproduzir LE 0.
    t_saida-necessidade_mp = 0.
  ELSE.
    t_saida-necessidade_mp = v_neces_mp.
  ENDIF.

  t_saida-materia_mp = t_saida-matnr.

* Saldo da Matéria_prima

  CLEAR: vl_tot_neces_mp, vl_tot_neces_mp.
  LOOP AT t_saida INTO wa_saida WHERE materia_mp EQ t_saida-matnr
                                  AND werks      EQ t_saida-werks.
    vl_tot_neces_mp = vl_tot_neces_mp + wa_saida-necessidade_mp.
  ENDLOOP.
ENDFORM.                    " APURA_SALDO_MP
*&---------------------------------------------------------------------*
*&      Form  PROD_OUTRO_CENTRO
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM prod_outro_centro .

  t_saida-op = t_saida-op + t_dad_rel-qte_or.

* Verifica o status da OP
  SELECT SINGLE idat2 ftrmi
    INTO (caufv-idat2, caufv-ftrmi)
    FROM caufv
    WHERE aufnr EQ t_dad_rel-or_pro.
  IF sy-subrc EQ 0.
    IF caufv-idat2 = '00000000'.

      CLEAR t_zrnppt_prev.

      READ TABLE t_zrnppt_prev WITH KEY  werks  = t_dad_rel-werks
                                         kunnr  = t_saida-cod_cli
                                         matnr  = t_dad_rel-matnr
                                         codigo = t_dad_rel-codigo.

      IF caufv-ftrmi = '00000000'.
        t_saida-op_aber = t_saida-op_aber + t_dad_rel-qte_or.
        IF t_zrnppt_prev-peso_peca IS NOT INITIAL.
          t_saida-op_aber_pec = ( t_saida-op_aber * 1000 )  / t_zrnppt_prev-peso_peca.
          t_dad_rel-op_peca   = ( t_dad_rel-qte_or * 1000 ) / t_zrnppt_prev-peso_peca.
        ENDIF.
        t_dad_rel-op_aber = t_dad_rel-qte_or.
      ELSE.
        t_saida-op_lib = t_saida-op_lib + t_dad_rel-qte_or.
        IF t_zrnppt_prev-peso_peca IS NOT INITIAL.
          t_saida-op_lib_pec = ( t_saida-op_lib * 1000 )   / t_zrnppt_prev-peso_peca.
          t_dad_rel-op_peca  = ( t_dad_rel-qte_or * 1000 ) / t_zrnppt_prev-peso_peca.
        ENDIF.
        t_dad_rel-op_lib = t_dad_rel-qte_or.
      ENDIF.

      READ TABLE t_aufk  WITH KEY aufnr = t_dad_rel-or_pro.
      IF sy-subrc IS INITIAL.
        SELECT znumoa znumov zitemov UP TO 1 ROWS
          INTO (t_dad_rel-op, t_dad_rel-nr_ov, t_dad_rel-it_ov)
          FROM zrnppt_popa
          WHERE znumoa = t_aufk-cy_seqnr.
        ENDSELECT.
      ENDIF.

      MODIFY t_dad_rel INDEX v_tabix.
    ENDIF.
  ENDIF.

ENDFORM.                    " PROD_OUTRO_CENTRO
*&---------------------------------------------------------------------*
*&      Form  VERIFICA_TIPO_MOVIMENTO
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM verifica_tipo_movimento  USING    p_ekbe_matnr   TYPE ekbe-matnr
                                       p_ekbe_charg   TYPE ekbe-charg
                                       p_ekbe_belnr   TYPE ekbe-belnr
                                       p_ekbe_buzei   TYPE ekbe-buzei
                              CHANGING p_answer TYPE c.

  CLEAR: wa_mseg_log.
  SELECT SINGLE *
    INTO wa_mseg_log
    FROM mseg
   WHERE matnr = p_ekbe_matnr
     AND charg = p_ekbe_charg
     AND smbln = p_ekbe_belnr
     AND smblp = p_ekbe_buzei
     AND bwart IN ('102','106').

  IF sy-subrc IS INITIAL.
    p_answer = 'X'.
  ENDIF.

ENDFORM.                    " VERIFICA_TIPO_MOVIMENTO
*&---------------------------------------------------------------------*
*&      Form  VERIFICA_OPCAO
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM verifica_opcao  CHANGING p_option TYPE c.
  CLEAR: v_answer.
  CALL FUNCTION 'POPUP_TO_CONFIRM'
    EXPORTING
      titlebar              = 'CONSULTA ESTOQUE MP'
      text_question         = 'ESCOLHA O TIPO DE RELATORIO:'
      text_button_1         = 'MMBE'
      text_button_2         = 'ZA09A'
      display_cancel_button = ''
      start_column          = 25
      start_row             = 6
      iv_quickinfo_button_1 = 'ACESSO A TRANSAÇÃO MMBE'
      iv_quickinfo_button_2 = 'ACESSO A TRANSAÇÃO ZA09A'
    IMPORTING
      answer                = v_answer.
  IF sy-subrc <> 0.
* Implement suitable error handling here
  ENDIF.

  p_option = v_answer.

ENDFORM.                    " VERIFICA_OPCAO
*&---------------------------------------------------------------------*
*&      Form  F_CARREGA_OVS_EMITIDAS
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM f_carrega_ovs_emitidas .

  TYPES: BEGIN OF ty_vbak_mapa,
           vbeln TYPE vbak-vbeln,
           auart TYPE vbak-auart,
           vkorg TYPE vbak-vkorg,
           vtweg TYPE vbak-vtweg,
           kunnr TYPE vbak-kunnr,
         END OF ty_vbak_mapa,

         BEGIN OF ty_vbap_mapa,
           vbeln TYPE vbap-vbeln,
           posnr TYPE vbap-posnr,
           werks TYPE vbap-matnr,
           matnr TYPE vbap-matnr,
           aufnr TYPE vbap-aufnr,
           untto TYPE vbap-untto,
           prodh TYPE vbap-prodh,
           spart TYPE vbap-spart,
           kweng TYPE vbap-kwmeng,
           meins TYPE vbap-meins,
           erdat TYPE vbap-erdat,
           erzet TYPE vbap-erzet,
           ntgew TYPE vbap-ntgew,
           cuobj TYPE vbap-cuobj,
         END OF ty_vbap_mapa,

         BEGIN OF ty_vbep_mapa,
           vbeln TYPE vbep-vbeln,
           posnr TYPE vbep-posnr,
           etenr TYPE vbep-etenr,
           wmeng TYPE vbep-wmeng,
           edatu TYPE vbep-edatu,
           lifsp TYPE vbep-lifsp,
         END OF ty_vbep_mapa.

  DATA: t_vbak_mapa TYPE TABLE OF ty_vbak_mapa,
        t_vbap_mapa TYPE TABLE OF ty_vbap_mapa,
        t_vbep_mapa TYPE TABLE OF ty_vbep_mapa.

  DATA: wa_vbak_mapa TYPE ty_vbak_mapa,
        wa_vbap_mapa TYPE ty_vbap_mapa,
        wa_vbep_mapa TYPE ty_vbep_mapa.

****Inicio - Fabio - 14.08.2020 - RDM 238614
*  DATA: BEGIN OF t_vbep_vbap_vbbe_marc OCCURS 0,
*    werks          LIKE vbap-werks,
*    matnr          LIKE vbap-matnr,
*    vbeln          LIKE vbep-vbeln,
*    posnr          LIKE vbep-posnr,
*    etenr          LIKE vbep-etenr,
*    wmeng          LIKE vbep-wmeng,
*    edatu          LIKE vbep-edatu,
*    lifsp          LIKE vbep-lifsp,
**  matnr          LIKE vbap-matnr,
*    aufnr          LIKE vbap-aufnr,
*    untto          LIKE vbap-untto,
*    spart          LIKE vbap-spart,
*    kwmeng         LIKE vbap-kwmeng,
*    meins          LIKE vbap-meins,
*    erdat          LIKE vbap-erdat,
*    erzet          LIKE vbap-erzet,
*    ntgew          LIKE vbap-ntgew,
*    cuobj          TYPE vbap-cuobj,
*    vkorg          LIKE vbak-vkorg,
*    vtweg          LIKE vbak-vtweg,
*    kunnr          LIKE vbak-kunnr,
*    auart          LIKE vbak-auart,
*    strgr          LIKE marc-strgr,
*    kdmat          LIKE knmt-kdmat,
*    codigo         TYPE ze_resul,
*    wrk02          LIKE vbap-werks,
*    last_reg(1)    TYPE c,
*  END OF t_vbep_vbap_vbbe_marc.
****Fim - Fabio - 14.08.2020 - RDM 238614

  CLEAR: t_vbak_mapa[], t_vbap_mapa[], t_vbep_mapa[].

  SELECT vbeln posnr etenr wmeng edatu lifsp
    INTO TABLE t_vbep_mapa
    FROM vbep
   WHERE edatu IN p_period.
  IF sy-subrc IS INITIAL.
    SELECT vbeln auart vkorg vtweg kunnr
      INTO TABLE t_vbak_mapa
      FROM vbak
      FOR ALL ENTRIES IN t_vbep_mapa
     WHERE vbeln  EQ t_vbep_mapa-vbeln
       AND auart  IN ('ZORB', 'ZOSV', 'ZXCS', 'ZSCS', 'ZSEM')
       AND augru   <> '14'
       AND augru   <> '64'
       AND kunnr  IN p_clt.
  ENDIF.

  IF t_vbak_mapa[] IS NOT INITIAL.
    SORT t_vbak_mapa BY vbeln.
    SELECT vbeln posnr werks matnr aufnr untto prodh spart kwmeng meins erdat erzet ntgew cuobj
     INTO TABLE t_vbap_mapa
     FROM vbap
      FOR ALL ENTRIES IN t_vbep_mapa
    WHERE vbeln  EQ t_vbep_mapa-vbeln
      AND posnr  EQ t_vbep_mapa-posnr
      AND matnr  IN p_mat
      AND werks  IN p_centro
      AND abgru  EQ ''.
  ENDIF.

**Preenche a tabela principal com os registros das tabelas auxiliares.
  SORT: t_vbep_mapa BY vbeln posnr.

  LOOP AT t_vbak_mapa INTO wa_vbak_mapa.
    LOOP AT t_vbap_mapa INTO wa_vbap_mapa WHERE vbeln = wa_vbak_mapa-vbeln.
      LOOP AT t_vbep_mapa INTO wa_vbep_mapa WHERE vbeln = wa_vbap_mapa-vbeln
                                              AND posnr = wa_vbap_mapa-posnr.

        t_vbep_vbap_vbbe_marc-vkorg  = wa_vbak_mapa-vkorg.
        t_vbep_vbap_vbbe_marc-vtweg  = wa_vbak_mapa-vtweg.
        t_vbep_vbap_vbbe_marc-kunnr  = wa_vbak_mapa-kunnr.
        t_vbep_vbap_vbbe_marc-vkorg  = wa_vbak_mapa-vkorg.
        t_vbep_vbap_vbbe_marc-auart  = wa_vbak_mapa-auart.
        t_vbep_vbap_vbbe_marc-werks  = wa_vbap_mapa-werks.
        t_vbep_vbap_vbbe_marc-matnr  = wa_vbap_mapa-matnr.
        t_vbep_vbap_vbbe_marc-aufnr  = wa_vbap_mapa-aufnr.
        t_vbep_vbap_vbbe_marc-meins  = wa_vbap_mapa-meins.
        t_vbep_vbap_vbbe_marc-untto  = wa_vbap_mapa-untto.
        t_vbep_vbap_vbbe_marc-spart  = wa_vbap_mapa-spart.
        t_vbep_vbap_vbbe_marc-erdat  = wa_vbap_mapa-erdat.
        t_vbep_vbap_vbbe_marc-erzet  = wa_vbap_mapa-erzet.
        t_vbep_vbap_vbbe_marc-ntgew  = wa_vbap_mapa-ntgew.
        t_vbep_vbap_vbbe_marc-vbeln  = wa_vbep_mapa-vbeln.
        t_vbep_vbap_vbbe_marc-posnr  = wa_vbep_mapa-posnr.
        t_vbep_vbap_vbbe_marc-etenr  = wa_vbep_mapa-etenr.
        t_vbep_vbap_vbbe_marc-wmeng  = wa_vbep_mapa-wmeng.
        t_vbep_vbap_vbbe_marc-kwmeng = wa_vbap_mapa-kweng.
        t_vbep_vbap_vbbe_marc-edatu  = wa_vbep_mapa-edatu.
        t_vbep_vbap_vbbe_marc-lifsp  = wa_vbep_mapa-lifsp.
        APPEND t_vbep_vbap_vbbe_marc.
      ENDLOOP.
    ENDLOOP.
  ENDLOOP.


*2. Totalizar as OVs Emitidas:
  LOOP AT t_saida.
    v_tabix = sy-tabix.
    LOOP AT t_vbep_vbap_vbbe_marc WHERE matnr = t_saida-matnr
                                  AND   werks = t_saida-werks.
      "      t_saida-ov_emitida = t_saida_prev-programa + t_vbep_vbap_vbbe_marc-wmeng.
***Inserção - Hermiro Carvalho - RDM 198272 - 28/06/2018 - Início
      IF t_saida-meins = 'PC'.
        t_saida-ov_emitida    = ( t_vbep_vbap_vbbe_marc-wmeng * ( t_saida-peso_peca / 1000 ) ) + t_saida-ov_emitida.
        t_saida-ov_emitida_pc = t_vbep_vbap_vbbe_marc-wmeng + t_saida-ov_emitida_pc.
      ELSE.
        t_saida-ov_emitida    = t_vbep_vbap_vbbe_marc-wmeng + t_saida-ov_emitida."Original mantida
        IF t_saida-peso_peca > 0.
          t_saida-ov_emitida_pc = ( t_vbep_vbap_vbbe_marc-wmeng / ( t_saida-peso_peca / 1000 ) ) + t_saida-ov_emitida_pc.
        ENDIF.
      ENDIF.
***Inserção - Hermiro Carvalho - RDM 198272 - 28/06/2018 - Fim
    ENDLOOP.
    MODIFY t_saida INDEX v_tabix.
  ENDLOOP.

***Inicio - Fabio - 14.08.2020 - RDM 238614
*Selecionar a posição de faturamento das OVs/ITEM/DIVISAÕ:
  IF t_vbep_vbap_vbbe_marc[] IS NOT INITIAL.
    SELECT *
      INTO TABLE it_entped_hd
      FROM zsuppt_entped_hd
      FOR ALL ENTRIES IN t_vbep_vbap_vbbe_marc
      WHERE werks  EQ t_vbep_vbap_vbbe_marc-werks
        AND vbeln  EQ t_vbep_vbap_vbbe_marc-vbeln
        AND posnr  EQ t_vbep_vbap_vbbe_marc-posnr.
    IF sy-subrc IS INITIAL.
* Exclusão das remessas já faturadas:
      LOOP AT t_vbep_vbap_vbbe_marc.

        v_tabix = sy-tabix.

        READ TABLE it_entped_hd WITH KEY
                   werks = t_vbep_vbap_vbbe_marc-werks
                   vbeln = t_vbep_vbap_vbbe_marc-vbeln
                   posnr = t_vbep_vbap_vbbe_marc-posnr
                   etenr = t_vbep_vbap_vbbe_marc-etenr.
        IF sy-subrc IS INITIAL.
          IF it_entped_hd-saldo LE 0.
            DELETE t_vbep_vbap_vbbe_marc INDEX v_tabix.
          ENDIF.
        ENDIF.
      ENDLOOP.

    ENDIF.

*Ordenação da tabela interna t_vbep_vbap_vbbe_marc:
    SORT t_vbep_vbap_vbbe_marc BY matnr werks edatu vbeln posnr.
  ENDIF.
***Fim - Fabio - 14.08.2020 - RDM 238614

ENDFORM.                    " F_CARREGA_OVS_EMITIDAS
*&---------------------------------------------------------------------*
*&      Form  SKU
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM sku USING p_value.

  DATA: BEGIN OF t_sku OCCURS 0,
          werks   TYPE t001w-werks,
          matnr   TYPE mara-matnr,
          idnrk   TYPE mara-matnr,
          roteiro TYPE char30,
        END OF t_sku.

  READ TABLE t_saida_aux INDEX 1.

  IF sy-subrc <> 0.
    MESSAGE text-015 TYPE c_e.
  ENDIF.

  REFRESH: t_sku.

  IF p_value IS INITIAL.
    p_value = t_saida-materia_prima.
  ENDIF.

  LOOP AT t_saida WHERE materia_prima = p_value.
    t_sku-werks = t_saida-werks.
    t_sku-matnr = t_saida-matnr.
    t_sku-idnrk = t_saida-materia_mp.
    t_sku-roteiro = t_saida-roteiro.
    APPEND t_sku.
  ENDLOOP.

  EXPORT t_sku TO MEMORY ID 'SKU'.

  CALL TRANSACTION 'ZPP_INTERFACE_PL' AND SKIP FIRST SCREEN.

ENDFORM.                    " SKU

INCLUDE zrnppr_planej_prod_visualizf01.
*&---------------------------------------------------------------------*
*&      Form  VERIFICA_PRIMEIRA_LINHA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM verifica_primeira_linha  USING p_centro      TYPE aufk-werks
                                    p_mat         TYPE zrnppt_prev-matnr
                                    p_period      TYPE sy-datum
                           CHANGING p_observacao  TYPE char30.
  DATA: t_srgbtbrel  TYPE TABLE OF srgbtbrel,
        wa_srgbtbrel TYPE srgbtbrel,
        wa_folder_id TYPE soodk,
        wa_object_id TYPE soodk,
        t_objcont    LIKE TABLE OF soli,
        t_hd_display LIKE	sood2,
        wa_objcont   LIKE soli,
        v_string     TYPE string,
        v_string2    TYPE string,
        v_count      TYPE i,
        v_instid2    TYPE srgbtbrel-instid_a.
  CLEAR: v_instid, t_srgbtbrel[], wa_srgbtbrel, t_objcont, t_hd_display,
         wa_folder_id, wa_object_id, wa_objcont, v_count,
         v_string, v_string2, v_instid2.
***Inserção - Hermiro Carvalho - RDM 201750 - 13/06/2018 - Início
  IF p_histob IS NOT INITIAL.
    CONCATENATE '%' p_centro p_mat '%' INTO v_instid2.
    SELECT *
      INTO TABLE t_srgbtbrel
      FROM srgbtbrel
     WHERE reltype  = 'NOTE'
       AND instid_a LIKE v_instid2
       AND typeid_a = 'GD'.
  ELSE.
***Inserção - Hermiro Carvalho - RDM 222245 - 03/07/2019 - Início
*    CONCATENATE p_centro p_mat p_period+4(2) p_period(4) INTO v_instid.
    CONCATENATE p_centro p_mat INTO v_instid.
***Inserção - Hermiro Carvalho - RDM 222245 - 03/07/2019 - Início
    SELECT SINGLE *
      INTO wa_srgbtbrel
      FROM srgbtbrel
     WHERE reltype  = 'NOTE'
       AND instid_a = v_instid
       AND typeid_a = 'GD'.
  ENDIF.

  IF sy-subrc IS INITIAL.
    IF p_histob IS NOT INITIAL."Verifica se opção pelo histórico esta marcada - Início
      SORT t_srgbtbrel BY instid_a.
      READ TABLE t_srgbtbrel INTO wa_srgbtbrel INDEX 1.
      wa_folder_id-objtp = wa_srgbtbrel-instid_b(3).
      wa_folder_id-objyr = wa_srgbtbrel-instid_b+3(2).
      wa_folder_id-objno = wa_srgbtbrel-instid_b+5(12).
      wa_object_id-objtp = wa_srgbtbrel-instid_b+17(3).
      wa_object_id-objyr = wa_srgbtbrel-instid_b+20(2).
      wa_object_id-objno = wa_srgbtbrel-instid_b+22(12).
      CONCATENATE p_centro p_mat wa_srgbtbrel-instid_a+12(2) wa_srgbtbrel-instid_a+14(4) INTO v_instid.
    ELSE."Opção pelo histórico não esta marcada
      wa_folder_id-objtp = wa_srgbtbrel-instid_b(3).
      wa_folder_id-objyr = wa_srgbtbrel-instid_b+3(2).
      wa_folder_id-objno = wa_srgbtbrel-instid_b+5(12).
      wa_object_id-objtp = wa_srgbtbrel-instid_b+17(3).
      wa_object_id-objyr = wa_srgbtbrel-instid_b+20(2).
      wa_object_id-objno = wa_srgbtbrel-instid_b+22(12).
    ENDIF."Verifica se opção pelo histórico esta marcada - Fim
***Inserção - Hermiro Carvalho - RDM 201750 - 13/06/2018 - Fim
    CALL FUNCTION 'SO_OBJECT_READ'
      EXPORTING
        folder_id                  = wa_folder_id
        object_id                  = wa_object_id
      IMPORTING
        object_hd_display          = t_hd_display
      TABLES
        objcont                    = t_objcont[]
      EXCEPTIONS
        active_user_not_exist      = 1
        communication_failure      = 2
        component_not_available    = 3
        folder_not_exist           = 4
        folder_no_authorization    = 5
        object_not_exist           = 6
        object_no_authorization    = 7
        operation_no_authorization = 8
        owner_not_exist            = 9
        parameter_error            = 10
        substitute_not_active      = 11
        substitute_not_defined     = 12
        system_failure             = 13
        x_error                    = 14
        OTHERS                     = 15.
    IF sy-subrc <> 0.
* Implement suitable error handling here
    ENDIF.
    IF t_objcont[] IS NOT INITIAL.
      READ TABLE t_objcont INTO wa_objcont INDEX 1.
      IF wa_objcont-line CS '##'."Verifica se existe mais de uma linha - Início
        SPLIT wa_objcont-line AT cl_abap_char_utilities=>newline INTO v_string v_string2."Existe mais de uma linha, então separa em 2 variáveis
        v_count = strlen( v_string )."Conta o tamanho da linha
        IF v_count <= 27."Verifica o tamanho da linha - Início
          CONCATENATE v_string '***' INTO p_observacao."Se for menor que 27 apenas concatena com asterisco
        ELSE."Se for mais de 30 caracteres, formata a saída com asterisco.
          CONCATENATE v_string(27) '***' INTO p_observacao.
        ENDIF."Verifica o tamanho da linha - Fim
      ELSE."Só existe uma linha
        SPLIT wa_objcont-line AT cl_abap_char_utilities=>newline INTO v_string v_string2.
        v_count = strlen( v_string )."Conta o tamanho da linha
        IF v_count <= 27."Verifica o tamanho da linha - Início
*          CONCATENATE wa_objcont-line '***' INTO p_observacao.
          p_observacao = v_string.
        ELSE."Se for mais de 30 caracteres, formata a saída com asterisco.
*          CONCATENATE wa_objcont-line(27) '***' INTO p_observacao.
          CONCATENATE v_string(27) '***' INTO p_observacao.
        ENDIF."Verifica o tamanho da linha - Fim
      ENDIF."Verifica se existe mais de uma linha - Fim
    ENDIF.
  ENDIF.
ENDFORM.                    " VERIFICA_PRIMEIRA_LINHA
*----------------------------------------------------------------------*
*  MODULE standard OUTPUT
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
MODULE standard OUTPUT.
  SET PF-STATUS 'STANDARD'.
  SET TITLEBAR 'TITLE_9150'.

ENDMODULE.                    "standard OUTPUT
*&---------------------------------------------------------------------*
*&      Form  f_get_color
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM f_get_color USING p1 p2 p3 p4.

  READ TABLE t_color INTO w_color WITH KEY fname = p2
                                           line  = p4.

  w_color-line     = p4.
  w_color-color    = p1.
  w_color-fname    = p2.
  w_color-nokeycol = p3.

  IF sy-subrc EQ 0.
    MODIFY t_color FROM w_color INDEX sy-tabix.
  ELSE.
    APPEND w_color TO t_color.
  ENDIF.

  CLEAR: w_color.

ENDFORM.                    "f_get_color
*&---------------------------------------------------------------------*
*&      Form  save_server
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM save_server .

  DATA: gv_file        TYPE rlgrap-filename,
        gv_file_others TYPE rlgrap-filename,
        lv_data        TYPE string,
        lv_color-col   TYPE c LENGTH 10.

*  CONCATENATE p_file(18) 'dwsu_gd_definition.txt 'INTO gv_file_others.
*  OPEN DATASET gv_file_others FOR OUTPUT IN TEXT MODE ENCODING DEFAULT.
*  IF sy-subrc NE 0.
*    MESSAGE 'Unable to create file' TYPE 'I'.
*    EXIT.
*  ENDIF.
*
*  LOOP AT field_definition.
*    IF sy-tabix = 1.
*      CONCATENATE 'LINE' 'FIELDNAME' 'REPTEXT_DDIC' 'OUTPUTLEN' INTO lv_data SEPARATED BY '|'.
*      TRANSFER lv_data TO gv_file_others.
*    ENDIF.
*    CONCATENATE field_definition-fieldname
*                field_definition-outputlen
*                field_definition-reptext_ddic INTO lv_data SEPARATED BY '|'.
*    TRANSFER lv_data TO gv_file_others.
*  ENDLOOP.
*  CLOSE DATASET gv_file_others.

* ----------------------------------------------------------------------
*  DATA: lv_controle TYPE int4.
*
*  lv_file_color = p_file.
*  REPLACE ALL OCCURRENCES OF '.txt' IN lv_file_color WITH '_'.
*  CONDENSE lv_file_color.
*
** 231951 RGC8096
*  CONCATENATE lv_file_color 'color.txt' INTO gv_file_others.
** CONCATENATE p_file(18) 'dwsu_gd_color.txt 'INTO gv_file_others.
*
*  OPEN DATASET gv_file_others FOR OUTPUT IN TEXT MODE ENCODING DEFAULT.
*  IF sy-subrc NE 0.
*    MESSAGE 'Unable to create file' TYPE 'I'.
*    EXIT.
*  ENDIF.
*
*  LOOP AT t_color.
*
*    CASE t_color-fname.
*
*      WHEN 'NIVEL_EST'.
*        lv_controle  = lv_controle + 1.
*        w_color-line = lv_controle.
*
*      WHEN OTHERS.
*        w_color-line = lv_controle.
*
*    ENDCASE.
*
*    lv_color-col = t_color-color.
*    IF sy-tabix = 1.
*      CONCATENATE 'LINE' 'FNAME' 'COLOR' 'NOKEYCOL' 'HEXADECIMAL' INTO lv_data SEPARATED BY '|'.
*      TRANSFER lv_data TO gv_file_others.
*    ENDIF.
*    CONCATENATE t_color-line
*                t_color-fname
*                lv_color-col
*                t_color-nokeycol
*                t_color-hexadecimal INTO lv_data SEPARATED BY '|'.
*    TRANSFER lv_data TO gv_file_others.
*  ENDLOOP.
*  CLOSE DATASET gv_file_others.
*
* ----------------------------------------------------------------------
  gv_file = p_file.

  OPEN DATASET gv_file IN TEXT MODE
                       ENCODING DEFAULT
                       FOR OUTPUT
                       IGNORING CONVERSION ERRORS.
  IF sy-subrc NE 0.
    MESSAGE 'Unable to create file' TYPE 'I'.
    EXIT.
  ENDIF.

* PERFORM build_header.
* lv_data = t_header-field.
* TRANSFER lv_data TO gv_file.


*-> RGC0419/S.U. 14/09/2020 - Início  - RDM 246412 ---------------------- *
**  PERFORM build_fields.

*  PERFORM zf_preparar_impressao.   "RDM324238(-)
*RDM324238 - Begin
  IF p_csv IS NOT INITIAL.
    PERFORM f_preparar_impressao_csv.

    LOOP AT t_texto_str.
      lv_data  = t_texto_str-field.
      TRANSFER lv_data TO gv_file.
    ENDLOOP.
  ELSE.
    PERFORM zf_preparar_impressao.

    LOOP AT t_texto.
      lv_data  = t_texto-field.
      TRANSFER lv_data TO gv_file.
    ENDLOOP.
  ENDIF.
*RDM324238 - End
*-> RGC0419/S.U. 14/09/2020 - Término - RDM 246412 ---------------------- *

*  lv_data  = v_string_3.
*  TRANSFER lv_data TO gv_file.
  CLOSE DATASET gv_file.
  MESSAGE 'Arquivo gravado com sucesso' TYPE 'S'.

ENDFORM.                    " SAVE_SERVER

*PP - Extrator dos dados rel. GD para PowerBI - RDM 246412
**&---------------------------------------------------------------------*
**&      Form  color_hexa
**&---------------------------------------------------------------------*
**       text
**----------------------------------------------------------------------*
*FORM color_hexa.
*
*  DATA: lv_controle TYPE int4.
*
*  LOOP AT t_color INTO w_color.
*
*    CASE w_color-color.
*
*      WHEN 0.
*        w_color-hexadecimal = '#E0FFFF'. "Col Background
*
*      WHEN 1.
*        w_color-hexadecimal = '#0000FF'. "Blue
*
*      WHEN 2.
*        w_color-hexadecimal = '#E0FFFF'. "Normal
*
*      WHEN 3.
*        w_color-hexadecimal = '#FFFF00'. "Yellow
*
*      WHEN 4.
*        w_color-hexadecimal = '#0000FF'. "Col Key Blue
*
*      WHEN 5.
*        w_color-hexadecimal = '#00FF00'. "Green
*
*      WHEN 6.
*        w_color-hexadecimal = '#FF0000'. "Red
*
*      WHEN 7.
*        w_color-hexadecimal = '#FF4500'. "Orange
*
*    ENDCASE.
*
*    MODIFY t_color FROM w_color INDEX sy-tabix.
*
*  ENDLOOP.
*
*ENDFORM.                    "color_hexa
*PP - Extrator dos dados rel. GD para PowerBI - RDM 246412

*&---------------------------------------------------------------------*
*&      Form  SAVE_LOCAL
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM save_local .

  lv_fname = p_file.

* PERFORM build_header.

*-> RGC0419/S.U. 14/09/2020 - Início  - RDM 246412 ---------------------- *
**  PERFORM build_fields.
*  PERFORM zf_preparar_impressao.   "RDM324238(-)
*RDM324238 - Begin
  IF p_csv IS NOT INITIAL.
    PERFORM f_preparar_impressao_csv.

    CALL FUNCTION 'GUI_DOWNLOAD'
      EXPORTING
        filename = lv_fname
        filetype = 'ASC'
        write_lf = 'X'
*       write_field_separator = '||'
      TABLES
        data_tab = t_texto_str " Pass the Output internal table
      EXCEPTIONS
        OTHERS   = 22.
  ELSE.
    PERFORM zf_preparar_impressao.

    CALL FUNCTION 'GUI_DOWNLOAD'
      EXPORTING
        filename = lv_fname
        filetype = 'ASC'
        write_lf = 'X'
*       write_field_separator = '||'
      TABLES
        data_tab = t_texto " Pass the Output internal table
      EXCEPTIONS
        OTHERS   = 22.
  ENDIF.
*RDM324238 - End
*-> RGC0419/S.U. 14/09/2020 - Término - RDM 246412 ---------------------- *
*
*  CALL FUNCTION 'GUI_DOWNLOAD'
*    EXPORTING
*      filename = lv_fname
*      filetype = 'ASC'
*      write_lf = 'X'
**     write_field_separator = '||'
*    TABLES
*      data_tab = t_texto " Pass the Output internal table
*    EXCEPTIONS
*      OTHERS   = 22.

ENDFORM.                    " SAVE_LOCAL
*&---------------------------------------------------------------------*
*&      Form  BUILD_HEADER
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM build_header.

  DATA: lv_field    TYPE string,
        lv_string   TYPE string,
        lv_string_1 TYPE string,
        lv_data     TYPE string.

  DATA: w_field_definition LIKE LINE OF field_definition,
        w_fields           LIKE LINE OF fields.

  LOOP AT field_definition.
    MOVE-CORRESPONDING field_definition TO fields.
    APPEND fields.
  ENDLOOP.

  FIELD-SYMBOLS: <fs>    LIKE LINE OF cl_abap_structdescr=>components,
                 <field> TYPE any.

  LOOP AT fields INTO w_fields.

    ol_str_line ?= cl_abap_typedescr=>describe_by_data( w_fields ).

    TRY.

        LOOP AT ol_str_line->components ASSIGNING <fs> .

          UNASSIGN <field>.
          ASSIGN COMPONENT <fs>-name OF STRUCTURE w_fields TO <field>.
          lv_field = <field>.
          CONCATENATE lv_string lv_field INTO lv_string SEPARATED BY '|'.

        ENDLOOP.

      CATCH cx_root.
        CLEAR: lv_string.

    ENDTRY.

  ENDLOOP.

  t_header-field = lv_string.
  APPEND t_header.

  CLEAR: w_field_definition, lv_string, lv_field.

  t_texto-field = t_header-field.
  APPEND t_texto.

ENDFORM.                    " BUILD_HEADER
**&---------------------------------------------------------------------*
**&      Form  BUILD_FIELDS
**&---------------------------------------------------------------------*
**       text
**----------------------------------------------------------------------*
**  -->  p1        text
**  <--  p2        text
**----------------------------------------------------------------------*
*FORM build_fields .
*
*  FIELD-SYMBOLS: <fs>    LIKE LINE OF cl_abap_structdescr=>components,
*                 <field> TYPE any.
*
*  CLEAR: lt_final, v_string_2.
*  LOOP AT t_final INTO lt_final.
*
*    IF v_string_3 IS INITIAL AND v_string_2 IS NOT INITIAL.
*      v_string_3 = v_string_2.
*    ENDIF.
*    ol_str_line ?= cl_abap_typedescr=>describe_by_data( lt_final ).
*
*    TRY.
*
*        LOOP AT ol_str_line->components ASSIGNING <fs> .
*
*          UNASSIGN <field>.
*          ASSIGN COMPONENT <fs>-name OF STRUCTURE lt_final TO <field>.
*          lv_field = <field>.
*
*          READ TABLE field_definition INTO w_field_definition WITH KEY fieldname = <fs>-name.
*
*          IF sy-subrc EQ 0.
*            CONCATENATE lv_string lv_field INTO lv_string SEPARATED BY '|'.
*            MOVE-CORRESPONDING w_field_definition TO w_definition.
*
*            READ TABLE t_definition WITH KEY fieldname = <fs>-name.
*            IF sy-subrc EQ 0.
*              MODIFY t_definition FROM w_definition INDEX sy-tabix.
*              CONCATENATE v_string_2 <fs>-name INTO v_string_2 SEPARATED BY '|'.
*            ELSE.
*              APPEND w_definition TO t_definition.
*              CONCATENATE v_string_2 <fs>-name INTO v_string_2 SEPARATED BY '|'.
*            ENDIF.
*          ENDIF.
*
*        ENDLOOP.
*
*      CATCH cx_root.
*        CLEAR: lv_string, v_string_2.
*
*    ENDTRY.
*
*    t_texto-field = lv_string.
*    APPEND t_texto.
*
*    CLEAR: lt_final, lv_string, lv_field.
*
*  ENDLOOP.
*
*  t_texto1-field = v_string_3.
*  APPEND t_texto1.
*
*  REFRESH: field_definition.
*  field_definition[] = t_definition[].
*
*  LOOP AT t_texto.
*    MOVE-CORRESPONDING t_texto TO t_texto1.
*    APPEND t_texto1.
*  ENDLOOP.
*
*  t_texto[] = t_texto1[].
*
*ENDFORM.                    " BUILD_FIELDS
*&---------------------------------------------------------------------*
*&      Form  PF_TESTE_DESENV
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM pf_teste_desenv .

  IF sy-uname = 'RGC8096'.

    t_saida-status_pcp = 'PCP'.
    t_saida-status_om  = 'OM'.
    t_saida-werks      = 'RN01'.
    APPEND t_saida.

  ENDIF.

ENDFORM.                    " PF_TESTE_DESENV
*&---------------------------------------------------------------------*
*&      Form  F_BUSCA_RESSUP_CLGE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM f_busca_ressup_clge .

  DATA: lt_saida_aux LIKE TABLE OF t_saida.

  lt_saida_aux[] = t_saida[].
  SORT lt_saida_aux BY werks
                       materia_prima.
  DELETE ADJACENT DUPLICATES FROM lt_saida_aux COMPARING werks materia_prima.

  IF NOT lt_saida_aux[] IS INITIAL.
    SELECT *
      FROM zsuppt_vsr_clge
      INTO TABLE it_vsr_clge
      FOR ALL ENTRIES IN lt_saida_aux
      WHERE centro  = lt_saida_aux-werks
        AND cd_mp   = lt_saida_aux-materia_prima.
    IF sy-subrc EQ 0.
      SORT it_vsr_clge.
    ENDIF.

  ENDIF.

ENDFORM.                    " F_BUSCA_RESSUP_CLGE


*-> RGC0419/S.U. 16/09/2020 - Início  - RDM 246412 ---------------------- *
*&---------------------------------------------------------------------*
*&      Form  ZF_PREPARAR_IMPRESSAO
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM zf_preparar_impressao .

  DATA lv_chave(34) TYPE c.

* Fazer cabeçalho conforme gt_fieldcat.
  CLEAR: t_texto, l_texto.
  REFRESH t_texto[].
  CONCATENATE

'CHAVE'
'||' 'MATERIAL'
'||' 'OBSERVAÇÃO'
**'||' 'Anex.'
'||' 'CENTRO'
'||' 'CENTRO PRODUTIVO'
'||' 'DESCRICAO MATERIAL'
'||' 'DEPÓSITO'
'||' 'CLIENTE'
'||' 'CÓDIGO CLIENTE'
'||' 'MATERIAL CLIENTE'
'||' 'PROJETO'
'||' 'PESO PC(KG)'
'||' '% REFUGO'
'||' 'FERRAMENTA'
'||' 'ROTEIRO'
'||' 'EMBALAGEM'
'||' 'QTD. RACKS'

'||' 'TAM. LOTE'
'||' 'ESTOQUE MIN.'
'||' 'ESTOQUE MIN. PC'
'||' 'ESTOQUE MIN. DIAS'
'||' 'PREVISÃO ORG'
'||' 'PREVISÃO ORG PC'
'||' 'DEMANDA EM OUTROS CENTRO'
'||' 'DEMANDA EM OUTROS CENTRO (PC)'
'||' 'OVs EMITIDAS'
'||' 'OVs EMITIDAS PC'
'||' 'PREVISÃO TN +1'
'||' 'PREVISÃO PC +1'
'||' 'PREVISÃO TN +2'
'||' 'PREVISÃO PC +2'
'||' 'PREVISÃO TN +3'
'||' 'PREVISÃO PC +3'
'||' 'PREVISÃO TN +4'
'||' 'PREVISÃO PC +4'

'||' 'PREVISÃO TN +5'
'||' 'PREVISÃO PC +5'
'||' 'A PRODUZIR'
'||' 'A PRODUZIR PC'
'||' 'PENDENCIA'
'||' 'PENDENCIA PC'
'||' 'OV'
'||' 'OV PC'
'||' 'PLANEJADAS TON'
'||' 'PLANEJADAS PC'
'||' 'DATA INICIO'
'||' 'VOLUME REM'
'||' 'VOLUME REM PC'
'||' 'VOLUME REM +1'
'||' 'VOLUME REM +1 PC'
**'||' 'VOLS. '

'||' 'REQUISIÇÕES TON'
'||' 'REQUISIÇÕES PC'
'||' 'PEDIDOS COMPRA TON'
'||' 'FORNECEDOR'
'||' 'PEDIDOS COMPRA PC'
'||' 'PEDIDOS COMPRA OUTRO CENTRO TON'
'||' 'PEDIDOS COMPRA OUTRO CENTRO PC'
'||' 'OP LIB.'
'||' 'OP LIB. PC'
'||' 'OP LIB. OUTRO CENTRO'
'||' 'OP LIB. OUTRO CENTRO PC'
'||' 'OP ABER.'
'||' 'OP ABER. PC'
'||' 'OP ABER. OUTRO CENTRO'
'||' 'OP ABER. OUTRO CENTRO PC'
'||' 'PRODUZIR (OP)'

'||' 'PRAZO'
'||' 'EXPEDICAO'
'||' 'EXPEDICAO PC'
'||' 'EXPEDICAO OUTRO CENTRO'
'||' 'EXPEDICAO OUTRO CENTRO PC'
'||' 'EXP.MAT.INTERM.'
'||' 'EXP.MAT.INTERM. PC'
'||' 'EXPEDICAO OUTRO CENTRO DET'
'||' 'MAT. CQ'
'||' 'MAT. CQ PC'
'||' 'FATURADO(DT REM)'
'||' 'FATURADO'
'||' 'FATURADO(DT REM) PC'
'||' 'FATURADO PC'
'||' 'FATURADO(DT REM) OUTRO CENTRO'
'||' 'FATURADO(DT REM) OUTRO CENTRO PC'
'||' 'FATURADO OUTRO CENTRO'

'||' 'FATURADO OUTRO CENTRO PC'
'||' 'PRODUCAO TOTAL'
'||' 'PRODUCAO TOTAL PC'
'||' 'NECESSIDADE MP'
'||' 'MATÉRIA PRIMA'
'||' 'DESCRIÇÃO MP'
'||' 'ESTOQUE MP'
'||' 'ESTOQUE MP OUTRO CENTRO'
'||' 'ESTOQUE MP (CONSIGNADO)'
'||' 'ESTOQUE MP (LISTA TÉC ANTIGA)'
'||' 'ESTOQUE MP (ALTERNATIVO)'
'||' 'ESTOQUE MP OUTRO CENTRO DET'
'||' 'SALDO MP'
'||' 'ANÁLISE PREV.'
'||' 'ANÁLISE PREV. PC'
'||' 'ANALISE'
'||' 'COBERTURA (DIAS)(TON)'

'||' 'COBERTURA (DIAS)(PC)'
'||' 'MÁX. EXPEDIÇÃO TON'
'||' 'MÁX. EXPEDIÇÃO PC'
'||' 'DATA ATUALIZAÇÂO'
'||' 'UN. MEDIDA MAT.'
'||' 'HIERARQ. MATER.'
'||' 'LOCAL ENTREGA'
'||' 'ATIVO'
'||' 'PRODUÇÃO'
'||' 'ESTOQUE NO FORNECEDOR'
'||' 'ESTOQUE NO FORNECEDOR (LISTA TÉC ANTIGA)'
'||' 'EXPEDICAO + FATURADO'
'||' 'VOLUME A PRODUZIR'
'||' 'VOLUME A PROGRAMAR'
'||' 'CHAMADAS AG. PROD.(PC)'
'||' 'CHAMADAS AG. PROD.(TN)'
'||' 'DATA NECES. CHAMADAS'
'||' 'ESTOQUE 1 DIA'
'||' 'COBERTURA Fat+Expedição ORG.'

'||' 'COBERTURA EXPEDIÇÃO ORG.'
'||' 'COBERTURA FATURADO ORG.'
'||' 'COBERTURA OP TOTAL DEMANDA ORG.'
'||' 'COBERTURA MP EM FORNECEDOR  DEMANDA ORG.'
'||' 'COBERTURA DE PEDIDOS COMPRA ORG.'
'||' 'COBERTURA EXPEDIÇÃO (60 DIAS)'
'||' 'COBERTURA Fat+Expedição (60 DIAS)'
'||' 'STATUS COBERTURA Expedição (60 DIAS)'
'||' 'STATUS COBERTURA Expedição ORG'
'||' 'VOLUME MENSAL ATENDIDO'
'||' 'SOMA DE MPS'
'||' 'EM TRANSFERÊNCIA'
'||' 'STATUS_PCP'
'||' 'MP USINA CHEGAR'
'||' 'DATA ENTR. USINA'

  INTO l_texto.

  APPEND l_texto TO t_texto.


* Mover para tabela auxiliar t_final_txt.
  CLEAR: t_final_txt, l_final_txt.
  REFRESH t_final_txt[].

*  LOOP AT t_final INTO l_final.
*    MOVE-CORRESPONDING l_final TO l_final_txt.
*    APPEND l_final_txt TO t_final_txt.
*  ENDLOOP.

  LOOP AT t_saida INTO l_saida.
    MOVE-CORRESPONDING l_saida TO l_saida_txt.
    APPEND l_saida_txt TO t_saida_txt.
  ENDLOOP.

* Mover as linhas da tabela t_saida com separador '||'.
*  LOOP AT t_final_txt INTO l_final_txt.
  LOOP AT t_saida_txt INTO l_saida_txt.

* Tratar os campos numéricos.
    WRITE:
        l_saida_txt-peso_peca               TO v_saida_txt_peso_peca,
        l_saida_txt-estoque_minimo          TO v_saida_txt_estoque_minimo,
        l_saida_txt-est_min_pec             TO v_saida_txt_est_min_pec,
        l_saida_txt-est_min_dias            TO v_saida_txt_est_min_dias,
        l_saida_txt-prev_org                TO v_saida_txt_prev_org,
        l_saida_txt-ov_emitida              TO v_saida_txt_ov_emitida,
        l_saida_txt-periodo_1               TO v_saida_txt_periodo_1,
        l_saida_txt-periodo_2               TO v_saida_txt_periodo_2,
        l_saida_txt-periodo_3               TO v_saida_txt_periodo_3,
        l_saida_txt-periodo_4               TO v_saida_txt_periodo_4,
        l_saida_txt-periodo_5               TO v_saida_txt_periodo_5,
        l_saida_txt-aproduzir               TO v_saida_txt_aproduzir,
        l_saida_txt-pendencia               TO v_saida_txt_pendencia,
        l_saida_txt-ov                      TO v_saida_txt_ov,
        l_saida_txt-planejadas              TO v_saida_txt_planejadas,
        l_saida_txt-volume_rem              TO v_saida_txt_volume_rem,
        l_saida_txt-volume_rem_1            TO v_saida_txt_volume_rem_1,
        l_saida_txt-requisicoes             TO v_saida_txt_requisicoes,
        l_saida_txt-pedidos                 TO v_saida_txt_pedidos,
        l_saida_txt-pedidos_pc              TO v_saida_txt_pedidos_pc,
        l_saida_txt-pedidos_cen             TO v_saida_txt_pedidos_cen,
        l_saida_txt-op_lib                  TO v_saida_txt_op_lib,
        l_saida_txt-op_lib_cen              TO v_saida_txt_op_lib_cen,
        l_saida_txt-op_lib_cen_pec          TO v_saida_txt_op_lib_cen_pec,
        l_saida_txt-op_aber                 TO v_saida_txt_op_aber,
        l_saida_txt-op_aber_cen             TO v_saida_txt_op_aber_cen,
        l_saida_txt-op_aber_cen_pec         TO v_saida_txt_op_aber_cen_pec,
        l_saida_txt-produzir_op             TO v_saida_txt_produzir_op,
        l_saida_txt-expedicao               TO v_saida_txt_expedicao,
        l_saida_txt-expedi_cen              TO v_saida_txt_expedi_cen,
        l_saida_txt-exp_inter               TO v_saida_txt_exp_inter,
        l_saida_txt-mat_cq                  TO v_saida_txt_mat_cq,
        l_saida_txt-faturado                TO v_saida_txt_faturado,
        l_saida_txt-faturado1               TO v_saida_txt_faturado1,
        l_saida_txt-faturado_cen            TO v_saida_txt_faturado_cen,
        l_saida_txt-faturado1_cen           TO v_saida_txt_faturado1_cen,
        l_saida_txt-prodtot                 TO v_saida_txt_prodtot,
        l_saida_txt-necessidade_mp          TO v_saida_txt_necessidade_mp,
        l_saida_txt-estoque_mp              TO v_saida_txt_estoque_mp,
        l_saida_txt-estoque_mp_cen          TO v_saida_txt_estoque_mp_cen,
        l_saida_txt-estoque_consignado      TO v_saida_txt_estoque_consignado,
        l_saida_txt-estoque_ltan            TO v_saida_txt_estoque_ltan,
        l_saida_txt-estoque_ltal            TO v_saida_txt_estoque_ltal,
        l_saida_txt-estoque_mp_cen_det      TO v_saida_txt_estoque_mp_cen_det,
        l_saida_txt-saldo_mp                TO v_saida_txt_saldo_mp,
        l_saida_txt-analise_prev            TO v_saida_txt_analise_prev,
        l_saida_txt-analise                 TO v_saida_txt_analise,
        l_saida_txt-max_expe_ton            TO v_saida_txt_max_expe_ton,
        l_saida_txt-max_expe_pc             TO v_saida_txt_max_expe_pc,
        l_saida_txt-estoque_forn            TO v_saida_txt_estoque_forn,
        l_saida_txt-estoque_forn_ltan       TO v_saida_txt_estoque_forn_ltan,
        l_saida_txt-estfat                  TO v_saida_txt_estfat,
        l_saida_txt-vol_aproduzir           TO v_saida_txt_vol_aproduzir,
        l_saida_txt-vol_aprogramar          TO v_saida_txt_vol_aprogramar,
        l_saida_txt-chmd_ag_prod_pc          TO v_saida_txt_chmd_ag_prod_pc,
        l_saida_txt-chmd_ag_prod_tn          TO v_saida_txt_chmd_ag_prod_tn,
        l_saida_txt-estoque1dia             TO v_saida_txt_estoque1dia,
        l_saida_txt-estqtran                TO v_saida_txt_estqtran,
        l_saida_txt-qtd_mp_che              TO v_saida_txt_qtd_mp_che,
        l_saida_txt-periodo_1pc             TO v_saida_txt_periodo_1pc,
        l_saida_txt-periodo_2pc             TO v_saida_txt_periodo_2pc,
        l_saida_txt-periodo_3pc             TO v_saida_txt_periodo_3pc,
        l_saida_txt-periodo_4pc             TO v_saida_txt_periodo_4pc,
        l_saida_txt-periodo_5pc             TO v_saida_txt_periodo_5pc,
        l_saida_txt-aproduzir_pec           TO v_saida_txt_aproduzir_pec,
        l_saida_txt-pendencia_pec           TO v_saida_txt_pendencia_pec,
        l_saida_txt-ov_peca                 TO v_saida_txt_ov_peca,
        l_saida_txt-planejadas_pc           TO v_saida_txt_planejadas_pc,
        l_saida_txt-volume_rem_pc           TO v_saida_txt_volume_rem_pc,
        l_saida_txt-volume_rem_1_pc         TO v_saida_txt_volume_rem_1_pc,
        l_saida_txt-requisicoes_pc          TO v_saida_txt_requisicoes_pc,
        l_saida_txt-pedidos_cen_pc          TO v_saida_txt_pedidos_cen_pc,
        l_saida_txt-op_lib_pec              TO v_saida_txt_op_lib_pec,
        l_saida_txt-op_aber_pec             TO v_saida_txt_op_aber_pec,
        l_saida_txt-expedicao_pec           TO v_saida_txt_expedicao_pec,
        l_saida_txt-expedi_cen_pc           TO v_saida_txt_expedi_cen_pc,
        l_saida_txt-exp_inter_pec           TO v_saida_txt_exp_inter_pec,
        l_saida_txt-mat_cq_pec              TO v_saida_txt_mat_cq_pec,
        l_saida_txt-faturado_pc             TO v_saida_txt_faturado_pc,
        l_saida_txt-faturado1_pc            TO v_saida_txt_faturado1_pc,
        l_saida_txt-faturado_cen_pc         TO v_saida_txt_faturado_cen_pc,
        l_saida_txt-faturado1_cen_pc        TO v_saida_txt_faturado1_cen_pc,
        l_saida_txt-prodtot_pec             TO v_saida_txt_prodtot_pec,
        l_saida_txt-analise_prev_pec        TO v_saida_txt_analise_prev_pec,
        l_saida_txt-abast                   TO v_saida_txt_abast,
        l_saida_txt-abast_pec               TO v_saida_txt_abast_pec,

        l_saida_txt-ausch                   TO v_saida_txt_ausch,
        l_saida_txt-qtd_rack                TO v_saida_txt_qtd_rack,
        l_saida_txt-prev_org_pec            TO v_saida_txt_prev_org_pec,
        l_saida_txt-prev_org_cen            TO v_saida_txt_prev_org_cen,
        l_saida_txt-prev_org_cen_pec        TO v_saida_txt_prev_org_cen_pec,
        l_saida_txt-ov_emitida_pc           TO v_saida_txt_ov_emitida_pc.

* Formatar campos de data.
*    CONCATENATE l_saida_txt-data_modif+6(2) '.' l_saida_txt-data_modif+4(2) '.' l_saida_txt-data_modif(4)
*           INTO v_saida_txt_data_modif.
*    CONCATENATE l_saida_txt-data_inicio+6(2) '.' l_saida_txt-data_inicio+4(2) '.' l_saida_txt-data_inicio(4)
*           INTO v_saida_txt_data_inicio.
    WRITE:  l_saida_txt-data_inicio  TO v_saida_txt_data_inicio,
            l_saida_txt-data_modif   TO v_saida_txt_data_modif,
            l_saida_txt-data_entre   TO v_saida_txt_data_entre,
            l_saida_txt-prazo        TO v_saida_txt_prazo,
            l_saida_txt-chmd_dt_neces        TO v_saida_txt_chmd_dt_neces.

* Formatar campo CHAVE.
    CONCATENATE l_saida_txt-matnr l_saida_txt-werks l_saida_txt-cod_cli INTO lv_chave SEPARATED BY space.

* Concatenar para a linha do t_texto.
    CONCATENATE
             lv_chave
        '||' l_saida_txt-matnr
        '||' l_saida_txt-observacao
**        '||' l_saida_txt-call_obs
        '||' l_saida_txt-werks
        '||' l_saida_txt-wrk02
        '||' l_saida_txt-descricao
        '||' l_saida_txt-lgort
        '||' l_saida_txt-cliente
        '||' l_saida_txt-cod_cli
        '||' l_saida_txt-codigo
        '||' l_saida_txt-projeto
        '||' v_saida_txt_peso_peca
        '||' v_saida_txt_ausch
        '||' l_saida_txt-ferramenta
        '||' l_saida_txt-roteiro
        '||' l_saida_txt-embalagem
        '||' v_saida_txt_qtd_rack

        '||' l_saida_txt-tamanho_lote
        '||' v_saida_txt_estoque_minimo
        '||' v_saida_txt_est_min_pec
        '||' v_saida_txt_est_min_dias
        '||' v_saida_txt_prev_org
        '||' v_saida_txt_prev_org_pec
        '||' v_saida_txt_prev_org_cen
        '||' v_saida_txt_prev_org_cen_pec
        '||' v_saida_txt_ov_emitida
        '||' v_saida_txt_ov_emitida_pc
        '||' v_saida_txt_periodo_1
        '||' v_saida_txt_periodo_1pc
        '||' v_saida_txt_periodo_2
        '||' v_saida_txt_periodo_2pc
        '||' v_saida_txt_periodo_3
        '||' v_saida_txt_periodo_3pc
        '||' v_saida_txt_periodo_4
        '||' v_saida_txt_periodo_4pc

        '||' v_saida_txt_periodo_5
        '||' v_saida_txt_periodo_5pc
        '||' v_saida_txt_aproduzir
        '||' v_saida_txt_aproduzir_pec
        '||' v_saida_txt_pendencia
        '||' v_saida_txt_pendencia_pec
        '||' v_saida_txt_ov
        '||' v_saida_txt_ov_peca
        '||' v_saida_txt_planejadas
        '||' v_saida_txt_planejadas_pc

        '||' v_saida_txt_data_inicio

        '||' v_saida_txt_volume_rem
        '||' v_saida_txt_volume_rem_pc
        '||' v_saida_txt_volume_rem_1
        '||' v_saida_txt_volume_rem_1_pc
**        '||' l_saida_txt-volume_icon

        '||' v_saida_txt_requisicoes
        '||' v_saida_txt_requisicoes_pc
        '||' v_saida_txt_pedidos
        '||' l_saida_txt-fornecedor
        '||' v_saida_txt_pedidos_pc
        '||' v_saida_txt_pedidos_cen
        '||' v_saida_txt_pedidos_cen_pc
        '||' v_saida_txt_op_lib
        '||' v_saida_txt_op_lib_pec
        '||' v_saida_txt_op_lib_cen
        '||' v_saida_txt_op_lib_cen_pec
        '||' v_saida_txt_op_aber
        '||' v_saida_txt_op_aber_pec
        '||' v_saida_txt_op_aber_cen
        '||' v_saida_txt_op_aber_cen_pec
        '||' v_saida_txt_produzir_op

        '||' v_saida_txt_prazo
        '||' v_saida_txt_expedicao
        '||' v_saida_txt_expedicao_pec
        '||' v_saida_txt_expedi_cen
        '||' v_saida_txt_expedi_cen_pc
        '||' v_saida_txt_exp_inter
        '||' v_saida_txt_exp_inter_pec
        '||' l_saida_txt-expedi_cen_det
        '||' v_saida_txt_mat_cq
        '||' v_saida_txt_mat_cq_pec
        '||' v_saida_txt_faturado
        '||' v_saida_txt_faturado1
        '||' v_saida_txt_faturado_pc
        '||' v_saida_txt_faturado1_pc
        '||' v_saida_txt_faturado_cen
        '||' v_saida_txt_faturado_cen_pc
        '||' v_saida_txt_faturado1_cen

        '||' v_saida_txt_faturado1_cen_pc
        '||' v_saida_txt_prodtot
        '||' v_saida_txt_prodtot_pec
        '||' v_saida_txt_necessidade_mp
        '||' l_saida_txt-materia_prima
        '||' l_saida_txt-desc_mp
        '||' v_saida_txt_estoque_mp
        '||' v_saida_txt_estoque_mp_cen
        '||' v_saida_txt_estoque_consignado
        '||' v_saida_txt_estoque_ltan
        '||' v_saida_txt_estoque_ltal
        '||' v_saida_txt_estoque_mp_cen_det
        '||' v_saida_txt_saldo_mp
        '||' v_saida_txt_analise_prev
        '||' v_saida_txt_analise_prev_pec
        '||' v_saida_txt_analise
        '||' v_saida_txt_abast

        '||' v_saida_txt_abast_pec
        '||' v_saida_txt_max_expe_ton
        '||' v_saida_txt_max_expe_pc

        '||' v_saida_txt_data_modif

        '||' l_saida_txt-meins
        '||' l_saida_txt-h_material
        '||' l_saida_txt-l_entrega
        '||' l_saida_txt-ativo
        '||' l_saida_txt-producao
        '||' v_saida_txt_estoque_forn
        '||' v_saida_txt_estoque_forn_ltan
        '||' v_saida_txt_estfat
        '||' v_saida_txt_vol_aproduzir
        '||' v_saida_txt_vol_aprogramar
        '||' v_saida_txt_chmd_ag_prod_pc
        '||' v_saida_txt_chmd_ag_prod_tn
        '||' v_saida_txt_chmd_dt_neces
        '||' v_saida_txt_estoque1dia
        '||' l_saida_txt-cobfat_exp_org

        '||' l_saida_txt-cob_exp_org
        '||' l_saida_txt-cob_fat_org
        '||' l_saida_txt-cob_op_total_dem_org
        '||' l_saida_txt-cob_mp_forne_dem_org
        '||' l_saida_txt-cob_ped_comp_org
        '||' l_saida_txt-cob_exp_60
        '||' l_saida_txt-cobfat_exp_60
        '||' l_saida_txt-cob_stat_exp_60
        '||' l_saida_txt-status_cobert_exp_org
        '||' l_saida_txt-vol_mensal_atendido
        '||' l_saida_txt-soma_mps
        '||' v_saida_txt_estqtran
        '||' l_saida_txt-status_pcp
        '||' v_saida_txt_qtd_mp_che
        '||' v_saida_txt_data_entre

    INTO l_texto.
    APPEND l_texto TO t_texto.

    CLEAR l_texto.

  ENDLOOP.

ENDFORM.                    " ZF_PREPARAR_IMPRESSAO
*-> RGC0419/S.U. 16/09/2020 - Término - RDM 246412 ---------------------- *



*-> RGC0419/S.U. 16/09/2020 - Início  - RDM 246412 ---------------------- *
*-> RGC0419/S.U. 16/09/2020 - Término - RDM 246412 ---------------------- *
*&---------------------------------------------------------------------*
*&      Form  F_MONTA_SAIDA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM f_monta_saida .
***Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Início
  DATA: wa_mslb     TYPE mslb,
        wa_mslb_aux TYPE mslb.
***Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Fim

*PP - Cenário Estoque Consignado - RDM 239896
  DATA: wa_consignado TYPE ty_consignado.
*PP - Cenário Estoque Consignado - RDM 239896

***Inserção - Hermiro Carvalho - 18/04/2016 - Início
  DATA: v_095   TYPE p DECIMALS 3 VALUE '0.095',
        v_005   TYPE p DECIMALS 3 VALUE '0.05',
***Inserção - Hermiro Carvalho - 18/04/2016 - Fim
        vl_ceex TYPE p DECIMALS 3,
        vl_cfat TYPE p DECIMALS 3,
        vl_cpdc TYPE p DECIMALS 3,
        vl_creq TYPE p DECIMALS 3,
        vl_ctot TYPE p DECIMALS 3.
  DATA: wa_pl_corte	TYPE zrnppt_pl_corte,
        wa_lfa1     TYPE lfa1.
  DATA: vl_log_mslb(80) TYPE c.

* >>> - 03.10.2019 14:06:46 - RDM 225838 - Simone
  DATA: l_period TYPE sy-datum.
* <<< - 03.10.2019 14:06:46 - RDM 225838 - Simone

  CLEAR: wa_mslb, t_mslb_aux[], t_mslb[].

  DATA: wa_color TYPE lvc_s_scol.
  "vl_index TYPE sy-tabix.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Início
  DATA: vl_op    TYPE zrnppt_prev-peso.
***Inserção - Hermiro Carvalho - RDM 105472 - 31/07/2014 - Fim

*PP - Cenário Estoque Consignado - RDM 239896
  DATA lt_werks_consignado TYPE STANDARD TABLE OF zsuget_variaveis WITH HEADER LINE.
*PP - Cenário Estoque Consignado - RDM 239896

***Inserção - Hermiro Carvalho - RDM 160097 - 12/04/2016 - Início
***Comentar - Hermiro Carvalho - RDM 160097 - 05/05/2016 - Início
  IF t_saida[] IS NOT INITIAL.
*    CLEAR: r_ptnumber[], wa_ptnumber.
*    LOOP AT t_saida INTO wa_saida.
*      wa_ptnumber-sign   = 'I'.   "I = include, E = exclude
*      wa_ptnumber-option = 'EQ'.  "EQ, BT, NE ....
*      wa_ptnumber-low    = wa_saida-codigo.
*      CONDENSE wa_ptnumber-low NO-GAPS.
*      APPEND wa_ptnumber TO r_ptnumber.
*      CLEAR: wa_saida, wa_ptnumber.
*    ENDLOOP.

    SELECT *
      INTO TABLE t_mslb
      FROM mslb
      FOR ALL ENTRIES IN t_saida
     WHERE matnr = t_saida-materia_prima
       AND werks = t_saida-werks
       AND lblab > 0.

    LOOP AT t_mslb INTO wa_mslb.

      vl_index = sy-tabix.

      CLEAR vl_log_mslb.

      PERFORM check_reserva_estoque_especial  USING wa_mslb-matnr
                                                    wa_mslb-werks
                                                    wa_mslb-charg
                                              CHANGING vl_log_mslb.
      IF vl_log_mslb IS NOT INITIAL.
        DELETE t_mslb INDEX vl_index.
      ENDIF.

    ENDLOOP.

*PP - Cenário Estoque Consignado - RDM 239896
    REFRESH: lt_werks_consignado,
             t_consignado.
    SELECT *
      FROM zsuget_variaveis
      INTO TABLE lt_werks_consignado
     WHERE repid  = 'ZSUMMI_EDINFE'
       AND field  = 'CONSIGNADOS'.
    IF lt_werks_consignado[] IS NOT INITIAL.

      CLEAR: r_werks_consignado[], r_werks_consignado.
      LOOP AT lt_werks_consignado.
        r_werks_consignado-sign   = 'I'.
        r_werks_consignado-option = 'EQ'.
        r_werks_consignado-low    = lt_werks_consignado-filtro.
        APPEND r_werks_consignado.
        CLEAR r_werks_consignado.
      ENDLOOP.

      SELECT matnr werks lgort charg ersda clabs
        INTO TABLE t_consignado
        FROM mchb
        FOR ALL ENTRIES IN t_saida
       WHERE matnr = t_saida-materia_prima
         AND werks IN r_werks_consignado
         AND clabs > 0.
    ENDIF.
*PP - Cenário Estoque Consignado - RDM 239896

***Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Início
    CLEAR: t_hist_lt[], t_mslb_ltan[], wa_mslb.
    SELECT werks matnr idnrk
      INTO TABLE t_hist_lt
      FROM zsuppt_hist_lt
       FOR ALL ENTRIES IN t_saida
     WHERE werks EQ t_saida-werks
      AND matnr  EQ t_saida-matnr.

    IF t_hist_lt[] IS NOT INITIAL.
      SELECT *
        INTO TABLE t_mslb_ltan
        FROM mslb
        FOR ALL ENTRIES IN t_hist_lt
       WHERE matnr = t_hist_lt-idnrk
         AND werks = t_hist_lt-werks
         AND lblab > 0.


      LOOP AT t_mslb_ltan INTO wa_mslb.
        vl_index = sy-tabix.
        CLEAR vl_log_mslb.
        PERFORM check_reserva_estoque_especial  USING wa_mslb-matnr
                                                      wa_mslb-werks
                                                      wa_mslb-charg
                                                CHANGING vl_log_mslb.
        IF vl_log_mslb IS NOT INITIAL.
          DELETE t_mslb_ltan INDEX vl_index.
        ENDIF.
      ENDLOOP.

    ENDIF.
***Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Fim

* >>> - 03.10.2019 14:06:46 - RDM 225838 - Simone
    CLEAR: l_period, t_estoqtransito, t_estoqtransito[].
    l_period = p_period-low - 30.
    IF p_matpri-low IS INITIAL.
      SELECT mblnr mjahr bwart matnr werks lgort charg menge budat_mkpf
             INTO TABLE t_estoqtransito
             FROM mseg
             FOR ALL ENTRIES IN t_saida
             WHERE   ( matnr = t_saida-matnr OR
                       matnr = t_saida-materia_mp )     AND
                     werks = t_saida-werks              AND
*  Início alteração LCS - RDM249282 - 09/11/2020
*                   ( bwart EQ '861' OR bwart EQ '862' ) AND
                   ( bwart EQ '861' OR bwart EQ '862' OR bwart EQ '641' ) AND
*  Fim alteração LCS - RDM249282 - 09/11/2020
                     budat_mkpf BETWEEN l_period AND p_period-high.
    ELSE.
      SELECT mblnr mjahr bwart matnr werks lgort charg menge budat_mkpf
             INTO TABLE t_estoqtransito
             FROM mseg
             FOR ALL ENTRIES IN t_saida
             WHERE   matnr = t_saida-materia_mp         AND
                     werks = t_saida-werks              AND
*  Início alteração LCS - RDM249282 - 09/11/2020
*                   ( bwart EQ '861' OR bwart EQ '862' ) AND
                   ( bwart EQ '861' OR bwart EQ '862' OR bwart EQ '641' ) AND
*  Fim alteração LCS - RDM249282 - 09/11/2020
                     budat_mkpf BETWEEN l_period AND p_period-high.
    ENDIF.

    IF sy-subrc EQ 0.
      SORT t_estoqtransito BY matnr bwart.
    ENDIF.
* <<< - 03.10.2019 14:06:46 - RDM 225838 - Simone
  ENDIF.
***Comentar - Hermiro Carvalho - RDM 160097 - 05/05/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 160097 - 12/04/2016 - Fim

  PERFORM f_busca_ressup_clge.

*RDM322490 - GVCA20241122 - Begin
  DATA(lt_matnr_gd) = t_saida[].

  SORT lt_matnr_gd BY matnr.
  DELETE ADJACENT DUPLICATES FROM lt_matnr_gd COMPARING matnr.

  IF lt_matnr_gd[] IS NOT INITIAL.
    SELECT matnr,
           dt_ruptura,
           tp_critico
      INTO TABLE @DATA(lt_ic_critico)
      FROM zsdt_ic_critico
      FOR ALL ENTRIES IN @lt_matnr_gd
      WHERE matnr   = @lt_matnr_gd-matnr
        AND status  = 'A'.

    SORT lt_ic_critico BY matnr.

  ENDIF.
*RDM322490 - GVCA20241122 - End

***Inserção - LG - RDM 334177 - 15/04/2026 - Início - Pré-cálculo campos EF
  PERFORM f_pre_calcula_campos_ef.
***Inserção - LG - RDM 334177 - 15/04/2026 - Fim - Pré-cálculo campos EF

****Inserção - Hermiro Carvalho - RDM 105472 - 21/07/2014 - Início
  CLEAR: wa_saida, vl_index, vl_op.
  LOOP AT t_saida INTO wa_saida.
    vl_index = sy-tabix.

    IF p_inati IS NOT INITIAL."Verifica parâmetro de tela.
      CLEAR: vl_op.
      vl_op = wa_saida-op_lib + wa_saida-op_aber.
      IF vl_op > '0.000' OR wa_saida-expedicao > '0.000'."Verifica condição
        DELETE t_saida INDEX vl_index.
        CONTINUE.
      ENDIF."Verifica condição
    ENDIF."Verifica parâmetro de tela.
    CASE wa_saida-nivel_est.
      WHEN 'SEM MP'.
        wa_color-fname     = 'NIVEL_EST'.
        wa_color-color-col = '6'."Vermelho
        APPEND wa_color TO wa_saida-cellcolor.
        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.

      WHEN 'SEM MP C/ MIN'.
        wa_color-fname     = 'NIVEL_EST'.
        wa_color-color-col = '7'."Laranja
        APPEND wa_color TO wa_saida-cellcolor.
        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.

      WHEN 'SEM OP'.
        wa_color-fname     = 'NIVEL_EST'.
        wa_color-color-col = '1'."Seria Lilás, mas o sab não tem essa cor
        APPEND wa_color TO wa_saida-cellcolor.
        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.

      WHEN 'A PRODUZIR'.
        wa_color-fname     = 'NIVEL_EST'.
        wa_color-color-col = '3'."Amarelo
        APPEND wa_color TO wa_saida-cellcolor.
        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.

      WHEN 'MÍNIMO'.
        wa_color-fname     = 'NIVEL_EST'.
        wa_color-color-col = '5'."Amarelo
        APPEND wa_color TO wa_saida-cellcolor.
        MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.

    ENDCASE.
***Inserção - Hermiro Carvalho - RDM 160097 - 12/04/2016 - Início

** Estoque no Fornecedor (T_SAIDA-ESTOQUE_FORN)
    LOOP AT t_mslb INTO wa_mslb WHERE matnr = wa_saida-materia_prima AND
                                      werks = wa_saida-werks.
      wa_saida-estoque_forn = wa_mslb-lblab + wa_saida-estoque_forn.
    ENDLOOP.

*PP - Cenário Estoque Consignado - RDM 239896
** Estoque Consignado (T_SAIDA-ESTOQUE_CONSIGNADO)
    LOOP AT t_consignado INTO wa_consignado WHERE matnr = wa_saida-materia_prima.
      wa_saida-estoque_consignado = wa_consignado-clabs + wa_saida-estoque_consignado.
    ENDLOOP.
*PP - Cenário Estoque Consignado - RDM 239896

***Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Início
** Estoque no Fornecedor de LT Antiga (T_SAIDA-ESTOQUE_FORN_LTAN)
    LOOP AT t_hist_lt INTO wa_hist_lt WHERE mantr = wa_saida-matnr AND
                                            werks = wa_saida-werks.
      LOOP AT t_mslb_ltan INTO wa_mslb WHERE matnr = wa_hist_lt-idnrk AND
                                             werks = wa_hist_lt-werks.
        wa_saida-estoque_forn_ltan = wa_mslb-lblab + wa_saida-estoque_forn_ltan.
      ENDLOOP.
      CLEAR: wa_hist_lt, wa_mslb.
    ENDLOOP.
***Inserção - Hermiro Carvalho - RDM 181009 - 29/05/2017 - Fim

***Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Início
*    t_mslb_aux[] = t_mslb[].
*    DELETE t_mslb_aux WHERE matnr <> wa_saida-matnr
*                        AND werks <> wa_saida-werks.
*    IF t_ekko IS NOT INITIAL.
*      CLEAR: wa_ekko.
*      SORT t_ekko BY matnr werks.
*      READ TABLE t_ekko INTO wa_ekko WITH KEY matnr = wa_saida-matnr
*                                              werks = wa_saida-werks.
*      CASE sy-subrc."Verifica se encontrou registro na t_ekko - Início
*        WHEN 0."Encontrou
*          READ TABLE t_mslb_aux INTO wa_mslb_aux WITH KEY matnr = wa_ekko-matnr
*                                                          werks = wa_ekko-werks
*                                                          lifnr = wa_ekko-lifnr.
*          CASE sy-subrc."Verifica se encontrou registro na t_mslb_aux - Início
*            WHEN 0."Encontrou
*              IF wa_mslb_aux-mp_forn GT 0.
*                wa_saida-mp_forn = wa_mslb_aux-mp_forn.
*              ENDIF.
*            WHEN OTHERS."Não encontrou
*          ENDCASE."Verifica se encontrou registro na t_mslb_aux - Fim
*        WHEN OTHERS."Não encontrou
*      ENDCASE."Verifica se encontrou registro na t_ekko - Fim
*    ENDIF.
*    SORT: t_mslb_aux BY dt_atualiza DESCENDING.
*    LOOP AT t_mslb_aux INTO wa_mslb_aux."Verifica as datas - Início
*      CASE wa_mslb-dt_atualiza."Campo data estrutura auxiliar - Início
*        WHEN space."Quando for vazio
*          wa_mslb = wa_mslb_aux.
*        WHEN OTHERS."Quando estiver preenchido.
*          IF wa_mslb_aux-dt_atualiza < wa_mslb-dt_atualiza.
*            DELETE t_mslb_aux FROM wa_mslb_aux.
*          ENDIF.
*          IF wa_mslb_aux-dt_atualiza > wa_mslb-dt_atualiza.
*            wa_mslb = wa_mslb_aux.
*          ENDIF.
*          CONTINUE."Continua para o próximo registro
*      ENDCASE."Campo data estrutura auxiliar - Fim
*    ENDLOOP."Verifica as datas - Fim
*    wa_saida-expedicao_forn = wa_mslb-expedicao_forn.
*    wa_saida-mp_fis_forn    = wa_mslb-mp_forn.
*    wa_saida-processo_forn  = wa_mslb-processo_forn.
***Comentar - Hermiro Carvalho - RDM 160097 - 16/05/2016 - Início

** Estoque + Faturado (T_SAIDA-ESTFAT)
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Início
*    wa_saida-estfat         = wa_saida-expedicao + wa_saida-expedicao_forn +
*                              wa_saida-faturado1 + wa_saida-faturado1_cen.
    wa_saida-estfat         = wa_saida-expedicao + wa_saida-expedi_cen +
                              wa_saida-expedicao_forn + wa_saida-faturado1 +
                              wa_saida-faturado1_cen.
***Inserção - Hermiro Carvalho - RDM 160097 - 17/05/2016 - Fim
***Comentar - Hermiro Carvalho - RDM 160097 - 16/05/2016 - Fim
***Comentar - Hermiro Carvalho - RDM 160097 - 03/05/2016 - Fim


**Volume a produzir
    "wa_saida-volume_aproduzir = (t_saida-aproduzir)   (confirmar substituição da regra atual)
***Comentar - Hermiro Carvalho - RDM 160097 - 18/04/2016 - Início
*    wa_saida-vol_aproduzir = 1.
*    IF ( v_095 * wa_saida-prev_org ) > wa_saida-estfat.
**    wa_saida-volume_aproduzir = wa_saida-prev_org - "coluna Estoque + Faturado" + wa_saida-estoque_minimo.
*      wa_saida-vol_aproduzir = wa_saida-prev_org + wa_saida-estoque_minimo.
*    ELSE.
*      IF ( wa_saida-prev_org + wa_saida-expedicao_forn ) > wa_saida-estoque_minimo.
*        wa_saida-vol_aproduzir = 0.
*      ELSE.
*        wa_saida-vol_aproduzir = wa_saida-estoque_minimo.
*      ENDIF.
*    ENDIF.
***Comentar - Hermiro Carvalho - RDM 160097 - 18/04/2016 - Fim

*wa_saida-vol_aproduzir = wa_saida-aproduzir.    "Não está com a nova regra que foi desenhada no chamado RDM 160097

* RDM 216498 RGC8096 04Abr2019
    wa_saida-vol_aproduzir = wa_saida-prev_org - wa_saida-estfat.
* RDM 216498 RGC8096 04Abr2019

**Volume a programar

*    IF ( v_095 * ( wa_saida-vol_aproduzir - wa_saida-requisicoes - wa_saida-pedidos ) < v_005 ).
****Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Início
**      wa_saida-vol_aproduzir = 0.
*      wa_saida-vol_aprogramar = 0.
****Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Fim
*    ELSE.
*      IF ( ( wa_saida-vol_aproduzir - wa_saida-requisicoes - wa_saida-pedidos ) < wa_saida-estoque_minimo ).
*        wa_saida-vol_aprogramar = wa_saida-estoque_minimo.
*      ELSE.
*        wa_saida-vol_aprogramar = wa_saida-vol_aproduzir - wa_saida-requisicoes - wa_saida-pedidos.
*      ENDIF.
*    ENDIF.

* RDM 216498 RGC8096 04Abr2019
    wa_saida-vol_aprogramar = wa_saida-prev_org -
    ( wa_saida-estfat + wa_saida-op_lib +
      wa_saida-op_aber + wa_saida-op_lib_cen +
        wa_saida-op_aber_cen ).
* RDM 216498 RGC8096 04Abr2019

**Estoque 1 dia
*      wa_saida-estoque1dia = wa_saida-estoque_minimo / (parâmetro da nova tabela de estoque mínimo).
***Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Início
*    IF wa_saida-estoque_minimo > 0.
*      wa_saida-estoque1dia = wa_saida-estoque_minimo / 1.
*    ENDIF.
*    SELECT SINGLE matnr werks rwpro
*      INTO wa_marc
*      FROM marc
*     WHERE matnr = wa_saida-matnr
*       AND werks = wa_saida-werks.
*
*    SELECT SINGLE rwpro mirwx
*      INTO wa_t438u
*      FROM t438u
*     WHERE rwpro EQ wa_marc-rwpro.
*    IF wa_t438u-mirwx > 0.
**      wa_saida-estoque1dia = wa_saida-estoque_minimo / wa_t438u-mirwx.
*      wa_saida-est_min_dias = wa_t438u-mirwx.
*      wa_saida-estoque1dia = wa_saida-estoque_minimo / wa_t438u-mirwx.
*    ENDIF.
***Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Fim
****Inserção - Sandra H. - RDM 174627 - 18/01/2017 - Início
    IF wa_saida-est_min_dias > 0.
      wa_saida-estoque1dia = wa_saida-estoque_minimo / wa_saida-est_min_dias.
    ENDIF.
****Inserção - Sandra H. - RDM 174627 - 18/01/2017 - FIM

** Processo Fornecedor (T_SAIDA-PROCESSO_FORN)
** <pendente a criação da tabela por parte do modulo de MM>

**Cobertura Estoque Expedição
    CASE wa_saida-prev_org.
      WHEN 0.
        wa_saida-ceex      = 'SEM DEMANDA'.
***Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Início
        wa_color-fname     = 'CEEX'.
        wa_color-color-col = '5'."Verde
        PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
***Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Fim
      WHEN OTHERS.
        IF wa_saida-estoque1dia > 0.
          CLEAR vl_ceex.
          vl_ceex = ( wa_saida-expedicao + wa_saida-expedicao_forn ) / wa_saida-estoque1dia.
          wa_saida-ceex = vl_ceex.
          CONDENSE wa_saida-ceex NO-GAPS.
          TRANSLATE wa_saida-ceex USING  '.,'.
***Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Início
          IF vl_ceex > 30.
            wa_color-fname     = 'CEEX'.
            wa_color-color-col = '5'."Verde
            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
          ENDIF.
          IF vl_ceex BETWEEN 15 AND 30.
            wa_color-fname     = 'CEEX'.
            wa_color-color-col = '3'."Amarelo
            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.

          ENDIF.
          IF vl_ceex < 15.
            wa_color-fname     = 'CEEX'.
            wa_color-color-col = '7'."Laranja
            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
          ENDIF.
          IF vl_ceex = 0.
            wa_color-fname     = 'CEEX'.
            wa_color-color-col = '6'."Vermelho
            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
          ENDIF.
          IF wa_saida-estoque_minimo = 0.
            wa_color-fname     = 'CEEX'.
            wa_color-color-col = '1'."Vermelho
            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
          ENDIF.
***Inserção - Hermiro Carvalho - RDM 160090 - 18/04/2016 - Fim
        ELSE.
          IF wa_saida-vol_aproduzir > 0.
            wa_saida-ceex = 'PRODUZIR SÓ DEMANDA'.
          ELSE.
            wa_saida-ceex = 'SEM DEMANDA'.
***Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Início
            wa_color-fname     = 'CEEX'.
            wa_color-color-col = '5'."Verde
            PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
***Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Fim
          ENDIF.
        ENDIF.
    ENDCASE.
***Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Início
    APPEND wa_color TO wa_saida-cellcolor.
    MODIFY t_saida FROM wa_saida INDEX vl_index TRANSPORTING cellcolor.
***Inserção - Hermiro Carvalho - RDM 160090 - 19/04/2016 - Fim

**Cobertura Faturado
    IF wa_saida-estoque1dia > 0.
      CLEAR vl_cfat.
      vl_cfat       = ( wa_saida-faturado1 + wa_saida-faturado1_cen ) / wa_saida-estoque1dia.
      wa_saida-cfat = vl_cfat.
      CONDENSE wa_saida-cfat NO-GAPS.
      TRANSLATE wa_saida-cfat USING  '.,'.
    ELSE.
      IF wa_saida-vol_aproduzir > 0.
        wa_saida-cfat = 'PRODUZIR SÓ DEMANDA'.
      ELSE.
        wa_saida-cfat = 'SEM DEMANDA'.
      ENDIF.
    ENDIF.

**Cobertura Pedidos Compras
    IF wa_saida-estoque1dia > 0.
      CLEAR vl_cpdc.
      vl_cpdc = ( wa_saida-pedidos - wa_saida-expedicao_forn ) / wa_saida-estoque1dia.
      wa_saida-cpdc = vl_cpdc.
      CONDENSE wa_saida-cpdc NO-GAPS.
      TRANSLATE wa_saida-cpdc USING  '.,'.
    ELSE.
      IF wa_saida-vol_aproduzir > 0.
        wa_saida-cpdc = 'PRODUZIR SÓ DEMANDA'.
      ELSE.
        wa_saida-cpdc = 'SEM DEMANDA'.
      ENDIF.
    ENDIF.


**Cobertura Requisições
    IF wa_saida-estoque1dia > 0.
      CLEAR vl_creq.
      vl_creq = wa_saida-requisicoes / wa_saida-estoque1dia.
      wa_saida-creq = vl_creq.
      CONDENSE wa_saida-creq NO-GAPS.
      TRANSLATE wa_saida-creq USING  '.,'.
    ELSE.
      IF wa_saida-vol_aproduzir > 0.
        wa_saida-creq = 'PRODUZIR SÓ DEMANDA'.
      ELSE.
        wa_saida-creq = 'SEM DEMANDA'.
      ENDIF.
    ENDIF.


**Cobertura Total
    IF wa_saida-creq = 'PRODUZIR SÓ DEMANDA'.
      wa_saida-ctot = 'PRODUZIR SÓ DEMANDA'.
    ELSE.
      IF wa_saida-creq = 'SEM DEMANDA'.
        wa_saida-ctot = 'SEM DEMANDA'.
      ELSE.
        CLEAR vl_ctot.
        vl_ctot = vl_ceex + vl_cfat + vl_cpdc + vl_creq.
        wa_saida-ctot = vl_ctot.
        CONDENSE wa_saida-ctot NO-GAPS.
        TRANSLATE wa_saida-ctot USING  '.,'.
      ENDIF.
    ENDIF.


**Status Cobertura Estoque Expedição
    IF wa_saida-creq = 'PRODUZIR SÓ DEMANDA'.
      wa_saida-scee = 'PRODUZIR SÓ DEMANDA'.
    ENDIF.

    IF wa_saida-scee IS INITIAL.
      IF wa_saida-creq = 'SEM DEMANDA'.
        wa_saida-scee = 'SEM DEMANDA'.
      ENDIF.
    ENDIF.

    IF wa_saida-scee IS INITIAL.
      IF vl_ceex < 7.
        wa_saida-scee = 'SEM ESTOQUE'.
      ENDIF.
    ENDIF.

    IF wa_saida-scee IS INITIAL.
      IF vl_ceex < 15.
        wa_saida-scee = 'MENOR DE 15 DIAS'.
      ENDIF.
    ENDIF.

    IF wa_saida-scee IS INITIAL.
      IF vl_ceex < 30.
        wa_saida-scee = 'ENTRE 15 e 30 DIAS'.
      ELSE.
        wa_saida-scee = 'ACIMA DE 30 DIAS'.
      ENDIF.
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 172372 - 01/12/2016 - Início
*    IF t_saida-materia_prima <> '*'.
    IF wa_saida-materia_prima <> '*'.
      PERFORM f_texto_material USING    wa_saida-materia_prima
                               CHANGING wa_saida-desc_mp.
    ENDIF.
***Inserção - Hermiro Carvalho - RDM 172372 - 01/12/2016 - Fim
***Inserção - Hermiro Carvalho - RDM 184185 - 18/09/2017 - Início
*    CLEAR: wa_pl_corte, wa_lfa1.
*    SELECT SINGLE *
*      INTO wa_pl_corte
*      FROM zrnppt_pl_corte
*     WHERE num_pl_corte = wa_saida-num_pl_corte.
*    IF sy-subrc IS INITIAL.
*      SELECT SINGLE *
*        INTO wa_lfa1
*        FROM lfa1
*       WHERE lifnr =  wa_pl_corte-lifnr.
*      IF sy-subrc IS INITIAL.
*        t_saida-sortl = wa_lfa1-sortl.
*      ENDIF.
*    ENDIF.
***Inserção - Hermiro Carvalho - RDM 184185 - 18/09/2017 - Fim
***Inserção - Hermiro Carvalho - RDM 196432 - 19/04/2018 - Início
    CLEAR: wa_saida-observacao.
*INC 2885055 - 31.07.19 - inicio
*    PERFORM verifica_primeira_linha USING p_centro-low wa_saida-matnr p_period-low CHANGING wa_saida-observacao.
    PERFORM verifica_primeira_linha USING wa_saida-werks wa_saida-matnr p_period-low CHANGING wa_saida-observacao.
*INC 2885055 - 31.07.19 - fim

***Inserção - Hermiro Carvalho - RDM 196641 - 06/08/2018 - Início
***Rotina para preenchimento dos novos campos de cobertura
    PERFORM calculo_cobertura USING wa_saida wa_color CHANGING wa_saida wa_saida-cellcolor.
***Inserção - Hermiro Carvalho - RDM 196641 - 06/08/2018 - Fim
***Inserção - Hermiro Carvalho - RDM 196432 - 19/04/2018 - Fim

    READ TABLE it_vsr_clge INTO wa_vsr_clge WITH KEY centro  = wa_saida-werks
                                                     cd_mp   = wa_saida-materia_prima.
    IF sy-subrc EQ 0.

      IF     wa_vsr_clge-peso_01 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_01.
        wa_saida-data_entre = wa_vsr_clge-data_01.
      ELSEIF wa_vsr_clge-peso_02 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_02.
        wa_saida-data_entre = wa_vsr_clge-data_02.
      ELSEIF wa_vsr_clge-peso_03 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_03.
        wa_saida-data_entre = wa_vsr_clge-data_03.
      ELSEIF wa_vsr_clge-peso_04 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_04.
        wa_saida-data_entre = wa_vsr_clge-data_04.
      ELSEIF wa_vsr_clge-peso_05 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_05.
        wa_saida-data_entre = wa_vsr_clge-data_05.
      ELSEIF wa_vsr_clge-peso_06 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_06.
        wa_saida-data_entre = wa_vsr_clge-data_06.
      ELSEIF wa_vsr_clge-peso_07 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_07.
        wa_saida-data_entre = wa_vsr_clge-data_07.
      ELSEIF wa_vsr_clge-peso_08 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_08.
        wa_saida-data_entre = wa_vsr_clge-data_08.
      ELSEIF wa_vsr_clge-peso_09 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_09.
        wa_saida-data_entre = wa_vsr_clge-data_09.
      ELSEIF wa_vsr_clge-peso_10 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_10.
        wa_saida-data_entre = wa_vsr_clge-data_10.
      ELSEIF wa_vsr_clge-peso_11 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_11.
        wa_saida-data_entre = wa_vsr_clge-data_11.
      ELSEIF wa_vsr_clge-peso_12 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_12.
        wa_saida-data_entre = wa_vsr_clge-data_12.
      ELSEIF wa_vsr_clge-peso_13 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_13.
        wa_saida-data_entre = wa_vsr_clge-data_13.
      ELSEIF wa_vsr_clge-peso_14 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_14.
        wa_saida-data_entre = wa_vsr_clge-data_14.
      ELSEIF wa_vsr_clge-peso_15 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_15.
        wa_saida-data_entre = wa_vsr_clge-data_15.
      ELSEIF wa_vsr_clge-peso_16 > 0.
        wa_saida-qtd_mp_che = wa_vsr_clge-peso_16.
        wa_saida-data_entre = wa_vsr_clge-data_16.
      ENDIF.
    ENDIF.

***Inicio - Fabio - 14.08.2020 - RDM 238614
    PERFORM f_ler_divisao_remessa USING wa_saida-data_inicio
                                        wa_saida-volume_rem
                                        wa_saida-volume_rem_pc
                                        wa_saida-volume_rem_1
                                        wa_saida-volume_rem_1_pc.

    WRITE '@B_TVAR@' CENTERED TO wa_saida-volume_icon.
***Fim - Fabio - 14.08.2020 - RDM 238614

*RDM322490 - GVCA20241122 - Begin
    "Status da Demanda
*   IF ( ( wa_saida-prev_org = 0 OR wa_saida-periodo_1 = 0 OR wa_saida-periodo_2 = 0 ) AND ( wa_saida-op_lib > 0 OR wa_saida-op_aber > 0 OR wa_saida-pedidos > 0 ) )
*      OR ( ( wa_saida-prev_org_cen = 0 ) AND (  wa_saida-pedidos_cen > 0 OR wa_saida-op_lib_cen > 0 OR wa_saida-op_aber_cen > 0 ) ).
*      wa_saida-status_dem = gc_status_dem_cancelada.

*RDM322490 - SANDRA H. 20241122 - INICIO - Se tiver ordens de produção ou pedidos e não houver demanda, status = Cancelada
    IF ( ( wa_saida-op_lib > 0 OR wa_saida-op_aber > 0 OR wa_saida-pedidos > 0 OR
           wa_saida-pedidos_cen > 0 OR wa_saida-op_lib_cen > 0 OR wa_saida-op_aber_cen > 0 ) AND
         ( wa_saida-prev_org = 0 AND wa_saida-periodo_1 = 0 AND wa_saida-periodo_2 = 0 AND wa_saida-prev_org_cen = 0 ) ).
      wa_saida-status_dem = gc_status_dem_cancelada.
*RDM322490 - SANDRA H. 20241122 - FIM

      wa_color-fname     = 'STATUS_DEM'.
      wa_color-color-col = '6'."Vermelho
      PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
      APPEND wa_color TO wa_saida-cellcolor.
    ELSE.
*RDM322490 - SANDRA H. 20241122 - INICIO - Se tiver ordens de produção ou pedidos e  houver demanda, status = CONFIRMADA
      IF ( ( wa_saida-op_lib > 0 OR wa_saida-op_aber > 0 OR wa_saida-pedidos > 0 OR
             wa_saida-pedidos_cen > 0 OR wa_saida-op_lib_cen > 0 OR wa_saida-op_aber_cen > 0 ) AND
           ( wa_saida-prev_org > 0 OR wa_saida-periodo_1 > 0 OR wa_saida-periodo_2 > 0 OR wa_saida-prev_org_cen > 0 ) ).
        wa_saida-status_dem = gc_status_dem_confirmada.
      ENDIF.

      IF ( ( wa_saida-op_lib = 0 AND wa_saida-op_aber = 0 AND wa_saida-pedidos = 0 AND
          wa_saida-pedidos_cen = 0 AND wa_saida-op_lib_cen > 0 AND wa_saida-op_aber_cen = 0 ) AND
        ( wa_saida-prev_org = 0 AND wa_saida-periodo_1 = 0 AND wa_saida-periodo_2 = 0 AND wa_saida-prev_org_cen = 0 ) ).

        wa_saida-status_dem = space. "BRANCO"
      ENDIF.
*RDM322490 - SANDRA H. 20241122 - FIM

    ENDIF.

    READ TABLE lt_ic_critico ASSIGNING FIELD-SYMBOL(<fs_ic_critico>) WITH KEY matnr = wa_saida-matnr BINARY SEARCH.
    IF sy-subrc = 0.
      wa_saida-ruptura_estq = <fs_ic_critico>-dt_ruptura.
* Início da Alteração - Nelson Junior - Stefanini - 18.08.2025
*      wa_saida-status_atend = gc_status_atend_mat_crit.

      SELECT SINGLE *
        FROM zsdt_ic_tp_critc
        INTO @DATA(ls_tp_critc)
        WHERE tp_critico EQ @<fs_ic_critico>-tp_critico.

      wa_saida-status_atend = ls_tp_critc-ds_tp_critico.
      CLEAR ls_tp_critc.
* Fim da Alteração - Nelson Junior - Stefanini - 18.08.2025
      wa_color-fname     = 'STATUS_ATEND'.
      wa_color-color-col = '6'."Vermelho
      PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
      APPEND wa_color TO wa_saida-cellcolor.
      "berm
    ENDIF.
*RDM322490 - GVCA20241122 - End

***Inserção - LG - RDM 334177 - 15/04/2026 - Início - Aplicação campos EF
    PERFORM f_aplica_campos_ef CHANGING wa_saida.

    IF wa_saida-zzacao = 'PCP Verificar saldo em produção: Item com demanda reduzida e/ou cancelada'.
      CLEAR: wa_color.
      wa_color-color-col  = 6."Vermelho
      wa_color-fname      = 'ZZACAO'.
      PERFORM f_get_color USING wa_color-color-col  wa_color-fname wa_color-nokeycol vl_index.
      APPEND wa_color TO wa_saida-cellcolor.

    ENDIF.
***Inserção - LG - RDM 334177 - 15/04/2026 - Fim - Aplicação campos EF

    wa_saida-line = vl_index.
    MODIFY t_saida FROM wa_saida INDEX vl_index.
***Inserção - Hermiro Carvalho - RDM 160097 - 12/04/2016 - Fim

*RDM 238614
    CLEAR wa_saida.
*RDM 238614

  ENDLOOP.

* >>> - 03.10.2019 14:06:46 - RDM 225838 - Simone
  CLEAR: wa_saida, vl_index.
  LOOP AT t_saida INTO wa_saida.
    vl_index = sy-tabix.
    READ TABLE t_estoqtransito WITH KEY matnr = wa_saida-matnr.
    IF sy-subrc EQ 0.
      PERFORM pesquisa_estoque_transito USING    wa_saida-matnr
                                        CHANGING wa_saida-estqtran.
    ELSE.
      PERFORM pesquisa_estoque_transito USING    wa_saida-materia_mp
                                        CHANGING wa_saida-estqtran.
    ENDIF.
    MODIFY t_saida FROM wa_saida INDEX vl_index.
  ENDLOOP.
* <<< - 03.10.2019 14:06:46 - RDM 225838 - Simone
********************************************************************************************
* Início da Alteração - Nelson Junior - Stefanini - 18.08.2025
  IF p_attmod EQ 'X'.
    PERFORM: zf_preenche_zppt_gd_modelos.
  ENDIF.
* Fim da Alteração - Nelson Junior - Stefanini - 18.08.2025
ENDFORM.

*INC 3708419
*&---------------------------------------------------------------------*
*&      Form  F_CHECK_DISPONIBILIDADE_LOTE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM f_check_disponibilidade_lote  USING    p_matnr
                                            p_werks
                                            p_lgort
                                            p_charg
                                   CHANGING p_clabs.

  DATA: lwa_ov_separa TYPE zrnppt_ov_separa.

  DATA: lit_resb TYPE STANDARD TABLE OF resb,
        lwa_resb TYPE resb.

  SELECT SINGLE *
    INTO lwa_ov_separa
    FROM zrnppt_ov_separa
    WHERE matnr = p_matnr
      AND werks = p_werks
      AND lgort = p_lgort
      AND charg = p_charg
      AND status = ''.
  IF sy-subrc IS INITIAL.
    CLEAR p_clabs.
  ENDIF.

  CHECK p_clabs GT 0.

  SELECT *
    INTO CORRESPONDING FIELDS OF TABLE lit_resb
    FROM resb
    WHERE xloek = ''
      AND matnr = p_matnr
      AND werks = p_werks
      AND lgort = p_lgort
      AND charg = p_charg
      AND bwart = '261'.
  LOOP AT lit_resb INTO lwa_resb.
    lwa_resb-bdmng = lwa_resb-bdmng - lwa_resb-enmng.
    IF lwa_resb-bdmng LT 0.
      CLEAR lwa_resb-bdmng.
    ENDIF.
    p_clabs = p_clabs - lwa_resb-bdmng.
  ENDLOOP.

  IF p_clabs LT 0.
    CLEAR p_clabs.
  ENDIF.

ENDFORM.
*INC 3708419

FORM user_command USING r_ucomm LIKE sy-ucomm
                        rs_field TYPE slis_selfield.

  DATA: lt_oest_log TYPE STANDARD TABLE OF zppt_oest_log,
        ls_fcat     TYPE lvc_s_fcat,
        lt_excl     TYPE slis_t_extab,
        ls_excl     TYPE slis_extab,
        gt_fieldcat TYPE slis_t_fieldcat_alv WITH HEADER LINE.

  DATA: lv_saldoest     TYPE  labst,
        lv_qtde_reserva TYPE  labst,
        lv_qtde_estoque TYPE  labst,
        lv_soma         TYPE  labst.


  IF rs_field-fieldname = 'LOTE_COMPART'.

    CLEAR: t_mchb_aux[], lt_fcat[], ls_fcat, t_hist_lt[],
           lt_excl[], ls_excl, gt_fieldcat[], lt_oest_log[], t_alternativa[].

***Seleciona dados na tabela zsuppt_lista_alt
    READ TABLE t_mchb_alv INTO wa_mchb_alv INDEX rs_field-tabindex.

    SELECT *
     INTO TABLE lt_oest_log
     FROM zppt_oest_log
     WHERE matnr    EQ wa_mchb_alv-matnr
       AND charg    EQ wa_mchb_alv-charg
       AND situacao EQ 'APROVADO'.

*   Stefanini/Elaine Omine - PRJ1372 - 15/08/2023 - Início
    IF sy-subrc EQ 0.
      SELECT werks, matnr, charg, matnr_pa
        FROM zsuppt_lista_alt
        INTO TABLE @DATA(tl_lista_alt)
         FOR ALL ENTRIES IN @lt_oest_log
       WHERE matnr      EQ @lt_oest_log-matnr
         AND charg      EQ @lt_oest_log-charg
         AND matnr_pa   EQ @lt_oest_log-matnr_apl
         AND desativada EQ @abap_true.
      IF sy-subrc EQ 0.
        LOOP AT tl_lista_alt INTO DATA(wl_lista_alt).
          "Elimina da visualização os registros eliminados
          DELETE lt_oest_log WHERE matnr     EQ wl_lista_alt-matnr
                               AND charg     EQ wl_lista_alt-charg
                               AND matnr_apl EQ wl_lista_alt-matnr_pa.
        ENDLOOP.
      ENDIF.
    ENDIF.
    IF lt_oest_log IS NOT INITIAL.
*   Stefanini/Elaine Omine - PRJ1372 - 15/08/2023 - Fim

      SELECT werks, matnr, charg, matnr_pa, rsnum, clabs
        INTO TABLE @DATA(tl_lista)
          FROM zsuppt_lista_alt
            FOR ALL ENTRIES IN @lt_oest_log
            WHERE matnr      EQ @lt_oest_log-matnr
              AND charg      EQ @lt_oest_log-charg
              AND matnr_pa   EQ @lt_oest_log-matnr_apl
              AND desativada EQ @abap_false.

      DATA(lt_oest_log_aux) = lt_oest_log[].
      SORT lt_oest_log_aux BY matnr.
      DELETE ADJACENT DUPLICATES FROM lt_oest_log_aux COMPARING matnr.

      IF lt_oest_log_aux[] IS NOT INITIAL.
        SELECT matnr, meins
          INTO TABLE @DATA(lt_mara)
            FROM mara
              FOR ALL ENTRIES IN @lt_oest_log_aux
              WHERE matnr = @lt_oest_log_aux-matnr.
      ENDIF.

      CLEAR v_menor_saldo.
      LOOP AT lt_oest_log ASSIGNING FIELD-SYMBOL(<fs_oest_log>).
        READ TABLE tl_lista INTO DATA(wl_lista)
          WITH KEY matnr = <fs_oest_log>-matnr
                   charg = <fs_oest_log>-charg
                   matnr_pa = <fs_oest_log>-matnr_apl.
        IF sy-subrc IS INITIAL.

          CALL FUNCTION 'ZPPF_SALDO_ALTERNATIVO'
            EXPORTING
              iv_matnr        = wl_lista-matnr
              iv_charg        = wl_lista-charg
              iv_matnt_apl    = wl_lista-matnr_pa
              iv_rsnum        = wl_lista-rsnum
            IMPORTING
              ev_saldoest     = lv_saldoest
              ev_qtde_reserva = lv_qtde_reserva
              ev_qtde_estoque = lv_qtde_estoque.

          lv_soma = lv_qtde_reserva + lv_qtde_estoque.
          v_menor_saldo = wl_lista-clabs - lv_soma.
          IF v_menor_saldo < 0.
            v_menor_saldo = 0.
          ENDIF.

          IF lv_saldoest < v_menor_saldo.
            v_menor_saldo = lv_saldoest.
          ENDIF.

          <fs_oest_log>-clabs = v_menor_saldo.

          TRY.
              DATA(ls_mara) = lt_mara[ matnr = wl_lista-matnr ].
              IF NOT ls_mara-meins = 'TO'.
                CALL FUNCTION 'MD_CONVERT_MATERIAL_UNIT'
                  EXPORTING
                    i_matnr              = ls_mara-matnr
                    i_in_me              = ls_mara-meins
                    i_out_me             = 'TO'
                    i_menge              = <fs_oest_log>-clabs
                  IMPORTING
                    e_menge              = <fs_oest_log>-clabs
                  EXCEPTIONS
                    error_in_application = 1
                    error                = 2
                    OTHERS               = 3.
              ENDIF.
            CATCH cx_sy_itab_line_not_found.
          ENDTRY.

        ENDIF.

      ENDLOOP.

      SORT lt_oest_log BY matnr charg matnr_apl werks.
      DELETE ADJACENT DUPLICATES FROM lt_oest_log COMPARING matnr charg matnr_apl werks.

      gt_fieldcat-fieldname = 'MATNR'.
      gt_fieldcat-seltext_m = 'Material'.
      APPEND gt_fieldcat.
      CLEAR: gt_fieldcat.

      gt_fieldcat-fieldname = 'WERKS'.
      gt_fieldcat-seltext_m = 'Centro'.
      APPEND gt_fieldcat.
      CLEAR: gt_fieldcat.

      gt_fieldcat-fieldname = 'LGORT'.
      gt_fieldcat-seltext_m = 'Depósito'.
      APPEND gt_fieldcat.
      CLEAR: gt_fieldcat.

      gt_fieldcat-fieldname = 'CHARG'.
      gt_fieldcat-seltext_m = 'Lote'.
      APPEND gt_fieldcat.
      CLEAR: gt_fieldcat.

      gt_fieldcat-fieldname = 'MATNR_APL'.
      gt_fieldcat-seltext_m = 'Mat.Receptor'.
      APPEND gt_fieldcat.
      CLEAR: gt_fieldcat.

      gt_fieldcat-fieldname = 'WERKS_APL'.
      gt_fieldcat-seltext_m = 'Centro Recept.'.
      APPEND gt_fieldcat.
      CLEAR: gt_fieldcat.

      gt_fieldcat-fieldname = 'LGORT_APL'.
      gt_fieldcat-seltext_m = 'Dep.Recept.'.
      APPEND gt_fieldcat.
      CLEAR: gt_fieldcat.

      gt_fieldcat-fieldname = 'CLABS'.
      gt_fieldcat-seltext_m = 'Peso'.
      APPEND gt_fieldcat.
      CLEAR: gt_fieldcat.

      ls_excl-fcode = '&ETA'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '%SC'.  APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '%SC+'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&OUP'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&ODN'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&ILT'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&OL0'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&CRB'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&CRL'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&CRR'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&CRE'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&AC1'. APPEND ls_excl TO lt_excl. " cancel
      ls_excl-fcode = '&ALL'. APPEND ls_excl TO lt_excl.
      ls_excl-fcode = '&SAL'. APPEND ls_excl TO lt_excl.

      CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
        EXPORTING
          i_grid_title          = 'Relatório Detalhado - Lote Compartilhado'
          i_structure_name      = 'LT_OEST_LOG'
          it_fieldcat           = gt_fieldcat[]
          it_excluding          = lt_excl
          i_callback_program    = sy-repid
          i_screen_start_column = 5
          i_screen_start_line   = 5
          i_screen_end_column   = 150
          i_screen_end_line     = 15
        TABLES
          t_outtab              = lt_oest_log
        EXCEPTIONS
          program_error         = 1
          OTHERS                = 2.
      IF sy-subrc <> 0.
        MESSAGE i000(0k) WITH sy-subrc.
      ENDIF.
    ELSE."Não encontrou registros
      MESSAGE 'Não foi encontrado registros' TYPE 'S' DISPLAY LIKE 'E'.
      RETURN.
    ENDIF.

  ENDIF.

ENDFORM.
*RDM324238 - Begin
*&---------------------------------------------------------------------*
*&      Form  F_PREPARAR_IMPRESSAO_CSV
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM f_preparar_impressao_csv .

  DATA lv_chave(34) TYPE c.

* Fazer cabeçalho conforme gt_fieldcat.
  CLEAR: t_texto_str, l_texto_str.
  REFRESH t_texto_str[].
  CONCATENATE

'CHAVE'
';' 'MATERIAL'
';' 'OBSERVAÇÃO'
**';' 'Anex.'
';' 'CENTRO'
';' 'CENTRO PRODUTIVO'
';' 'DESCRICAO MATERIAL'
';' 'DEPÓSITO'
';' 'CLIENTE'
';' 'CÓDIGO CLIENTE'
';' 'MATERIAL CLIENTE'
';' 'PROJETO'
';' 'PESO PC(KG)'
';' '% REFUGO'
';' 'FERRAMENTA'
';' 'ROTEIRO'
';' 'EMBALAGEM'
';' 'QTD. RACKS'

';' 'TAM. LOTE'
';' 'ESTOQUE MIN.'
';' 'ESTOQUE MIN. PC'
';' 'ESTOQUE MIN. DIAS'
';' 'PREVISÃO ORG'
';' 'PREVISÃO ORG PC'
';' 'DEMANDA EM OUTROS CENTRO'
';' 'DEMANDA EM OUTROS CENTRO (PC)'
';' 'OVs EMITIDAS'
';' 'OVs EMITIDAS PC'
';' 'PREVISÃO TN +1'
';' 'PREVISÃO PC +1'
';' 'PREVISÃO TN +2'
';' 'PREVISÃO PC +2'
';' 'PREVISÃO TN +3'
';' 'PREVISÃO PC +3'
';' 'PREVISÃO TN +4'
';' 'PREVISÃO PC +4'

';' 'PREVISÃO TN +5'
';' 'PREVISÃO PC +5'
';' 'A PRODUZIR'
';' 'A PRODUZIR PC'
';' 'PENDENCIA'
';' 'PENDENCIA PC'
';' 'OV'
';' 'OV PC'
';' 'PLANEJADAS TON'
';' 'PLANEJADAS PC'
';' 'DATA INICIO'
';' 'VOLUME REM'
';' 'VOLUME REM PC'
';' 'VOLUME REM +1'
';' 'VOLUME REM +1 PC'
**';' 'VOLS. '

';' 'REQUISIÇÕES TON'
';' 'REQUISIÇÕES PC'
';' 'PEDIDOS COMPRA TON'
';' 'FORNECEDOR'
';' 'PEDIDOS COMPRA PC'
';' 'PEDIDOS COMPRA OUTRO CENTRO TON'
';' 'PEDIDOS COMPRA OUTRO CENTRO PC'
';' 'OP LIB.'
';' 'OP LIB. PC'
';' 'OP LIB. OUTRO CENTRO'
';' 'OP LIB. OUTRO CENTRO PC'
';' 'OP ABER.'
';' 'OP ABER. PC'
';' 'OP ABER. OUTRO CENTRO'
';' 'OP ABER. OUTRO CENTRO PC'
';' 'PRODUZIR (OP)'

';' 'PRAZO'
';' 'EXPEDICAO'
';' 'EXPEDICAO PC'
';' 'EXPEDICAO OUTRO CENTRO'
';' 'EXPEDICAO OUTRO CENTRO PC'
';' 'EXP.MAT.INTERM.'
';' 'EXP.MAT.INTERM. PC'
';' 'EXPEDICAO OUTRO CENTRO DET'
';' 'MAT. CQ'
';' 'MAT. CQ PC'
';' 'FATURADO(DT REM)'
';' 'FATURADO'
';' 'FATURADO(DT REM) PC'
';' 'FATURADO PC'
';' 'FATURADO(DT REM) OUTRO CENTRO'
';' 'FATURADO(DT REM) OUTRO CENTRO PC'
';' 'FATURADO OUTRO CENTRO'

';' 'FATURADO OUTRO CENTRO PC'
';' 'PRODUCAO TOTAL'
';' 'PRODUCAO TOTAL PC'
';' 'NECESSIDADE MP'
';' 'MATÉRIA PRIMA'
';' 'DESCRIÇÃO MP'
';' 'ESTOQUE MP'
';' 'ESTOQUE MP OUTRO CENTRO'
';' 'ESTOQUE MP (CONSIGNADO)'
';' 'ESTOQUE MP (LISTA TÉC ANTIGA)'
';' 'ESTOQUE MP (ALTERNATIVO)'
';' 'ESTOQUE MP OUTRO CENTRO DET'
';' 'SALDO MP'
';' 'ANÁLISE PREV.'
';' 'ANÁLISE PREV. PC'
';' 'ANALISE'
';' 'COBERTURA (DIAS)(TON)'

';' 'COBERTURA (DIAS)(PC)'
';' 'MÁX. EXPEDIÇÃO TON'
';' 'MÁX. EXPEDIÇÃO PC'
';' 'DATA ATUALIZAÇÂO'
';' 'UN. MEDIDA MAT.'
';' 'HIERARQ. MATER.'
';' 'LOCAL ENTREGA'
';' 'ATIVO'
';' 'PRODUÇÃO'
';' 'ESTOQUE NO FORNECEDOR'
';' 'ESTOQUE NO FORNECEDOR (LISTA TÉC ANTIGA)'
';' 'EXPEDICAO + FATURADO'
';' 'VOLUME A PRODUZIR'
';' 'VOLUME A PROGRAMAR'
';' 'CHAMADAS AG. PROD.(PC)'
';' 'CHAMADAS AG. PROD.(TN)'
';' 'DATA NECES. CHAMADAS'
';' 'ESTOQUE 1 DIA'
';' 'COBERTURA Fat+Expedição ORG.'

';' 'COBERTURA EXPEDIÇÃO ORG.'
';' 'COBERTURA FATURADO ORG.'
';' 'COBERTURA OP TOTAL DEMANDA ORG.'
';' 'COBERTURA MP EM FORNECEDOR  DEMANDA ORG.'
';' 'COBERTURA DE PEDIDOS COMPRA ORG.'
';' 'COBERTURA EXPEDIÇÃO (60 DIAS)'
';' 'COBERTURA Fat+Expedição (60 DIAS)'
';' 'STATUS COBERTURA Expedição (60 DIAS)'
';' 'STATUS COBERTURA Expedição ORG'
';' 'VOLUME MENSAL ATENDIDO'
';' 'SOMA DE MPS'
';' 'EM TRANSFERÊNCIA'
';' 'STATUS_PCP'
';' 'MP USINA CHEGAR'
';' 'DATA ENTR. USINA'

  INTO l_texto_str.

*  "modo dinamico
*  DATA: ls_saida LIKE LINE OF t_saida.
*
*  FIELD-SYMBOLS: <fs_value>    TYPE any,
*                 <fs_variable> TYPE any.
*
*  DATA: lo_structdescr TYPE REF TO cl_abap_structdescr.
*
*  "constroi fieldcat para puxar descricoes de campos exatamente igual ao ALV
*  PERFORM f_monta_campos.
*
*  lo_structdescr ?= cl_abap_structdescr=>describe_by_data( ls_saida ).
*
*  DATA(lt_comp_tab) = lo_structdescr->get_components( ).

*  LOOP AT lt_comp_tab ASSIGNING FIELD-SYMBOL(<fs_component>).
*    DATA(lv_tabix) = sy-tabix.
*
*    READ TABLE gt_fieldcat ASSIGNING FIELD-SYMBOL(<fs_fcat>) WITH KEY fieldname = <fs_component>-name.

*  DATA: lr_ignore_fields TYPE RANGE OF fieldname.
*
*  lr_ignore_fields = VALUE #(
*    ( sign = 'I' option = 'EQ' low = 'CALL_OBS' )
*  ).
*
*  LOOP AT gt_fieldcat ASSIGNING FIELD-SYMBOL(<fs_fcat>).
*    DATA(lv_tabix) = sy-tabix.
*
*    READ TABLE lt_comp_tab ASSIGNING FIELD-SYMBOL(<fs_component>) WITH KEY name = <fs_fcat>-fieldname.
*
*    CHECK sy-subrc = 0 AND <fs_fcat>-fieldname NOT IN lr_ignore_fields.
*
*    IF lv_tabix = 1.
*      l_texto_str-field = <fs_fcat>-reptext_ddic.
*    ELSE.
*      l_texto_str-field = |{ l_texto_str-field };{ <fs_fcat>-reptext_ddic }|.
*    ENDIF.
*  ENDLOOP.
*
  APPEND l_texto_str TO t_texto_str.


* Mover para tabela auxiliar t_final_txt.
  CLEAR: t_final_txt, l_final_txt.
  REFRESH t_final_txt[].

  LOOP AT t_final INTO l_final.
    MOVE-CORRESPONDING l_final TO l_final_txt.
    APPEND l_final_txt TO t_final_txt.
  ENDLOOP.

  LOOP AT t_saida INTO l_saida.
    MOVE-CORRESPONDING l_saida TO l_saida_txt.
    APPEND l_saida_txt TO t_saida_txt.
  ENDLOOP.

* Mover as linhas da tabela t_saida com separador ';'.
*  LOOP AT t_final_txt INTO l_final_txt.
  LOOP AT t_saida_txt INTO l_saida_txt.

* Tratar os campos numéricos.
    WRITE:
        l_saida_txt-peso_peca               TO v_saida_txt_peso_peca,
        l_saida_txt-estoque_minimo          TO v_saida_txt_estoque_minimo,
        l_saida_txt-est_min_pec             TO v_saida_txt_est_min_pec,
        l_saida_txt-est_min_dias            TO v_saida_txt_est_min_dias,
        l_saida_txt-prev_org                TO v_saida_txt_prev_org,
        l_saida_txt-ov_emitida              TO v_saida_txt_ov_emitida,
        l_saida_txt-ov_emitida_pc           TO v_saida_txt_ov_emitida_pc,
        l_saida_txt-periodo_1               TO v_saida_txt_periodo_1,
        l_saida_txt-periodo_2               TO v_saida_txt_periodo_2,
        l_saida_txt-periodo_3               TO v_saida_txt_periodo_3,
        l_saida_txt-periodo_4               TO v_saida_txt_periodo_4,
        l_saida_txt-periodo_5               TO v_saida_txt_periodo_5,
        l_saida_txt-aproduzir               TO v_saida_txt_aproduzir,
        l_saida_txt-pendencia               TO v_saida_txt_pendencia,
        l_saida_txt-ov                      TO v_saida_txt_ov,
        l_saida_txt-planejadas              TO v_saida_txt_planejadas,
        l_saida_txt-volume_rem              TO v_saida_txt_volume_rem,
        l_saida_txt-volume_rem_1            TO v_saida_txt_volume_rem_1,
        l_saida_txt-requisicoes             TO v_saida_txt_requisicoes,
        l_saida_txt-pedidos                 TO v_saida_txt_pedidos,
        l_saida_txt-pedidos_pc              TO v_saida_txt_pedidos_pc,
        l_saida_txt-pedidos_cen             TO v_saida_txt_pedidos_cen,
        l_saida_txt-op_lib                  TO v_saida_txt_op_lib,
        l_saida_txt-op_lib_cen              TO v_saida_txt_op_lib_cen,
        l_saida_txt-op_lib_cen_pec          TO v_saida_txt_op_lib_cen_pec,
        l_saida_txt-op_aber                 TO v_saida_txt_op_aber,
        l_saida_txt-op_aber_cen             TO v_saida_txt_op_aber_cen,
        l_saida_txt-op_aber_cen_pec         TO v_saida_txt_op_aber_cen_pec,
        l_saida_txt-produzir_op             TO v_saida_txt_produzir_op,
        l_saida_txt-expedicao               TO v_saida_txt_expedicao,
        l_saida_txt-expedi_cen              TO v_saida_txt_expedi_cen,
        l_saida_txt-exp_inter               TO v_saida_txt_exp_inter,
        l_saida_txt-mat_cq                  TO v_saida_txt_mat_cq,
        l_saida_txt-faturado                TO v_saida_txt_faturado,
        l_saida_txt-faturado1               TO v_saida_txt_faturado1,
        l_saida_txt-faturado_cen            TO v_saida_txt_faturado_cen,
        l_saida_txt-faturado1_cen           TO v_saida_txt_faturado1_cen,
        l_saida_txt-prodtot                 TO v_saida_txt_prodtot,
        l_saida_txt-necessidade_mp          TO v_saida_txt_necessidade_mp,
        l_saida_txt-estoque_mp              TO v_saida_txt_estoque_mp,
        l_saida_txt-estoque_mp_cen          TO v_saida_txt_estoque_mp_cen,
        l_saida_txt-estoque_consignado      TO v_saida_txt_estoque_consignado,
        l_saida_txt-estoque_ltan            TO v_saida_txt_estoque_ltan,
        l_saida_txt-estoque_ltal            TO v_saida_txt_estoque_ltal,
        l_saida_txt-estoque_mp_cen_det      TO v_saida_txt_estoque_mp_cen_det,
        l_saida_txt-saldo_mp                TO v_saida_txt_saldo_mp,
        l_saida_txt-analise_prev            TO v_saida_txt_analise_prev,
        l_saida_txt-analise                 TO v_saida_txt_analise,
        l_saida_txt-max_expe_ton            TO v_saida_txt_max_expe_ton,
        l_saida_txt-max_expe_pc             TO v_saida_txt_max_expe_pc,
        l_saida_txt-estoque_forn            TO v_saida_txt_estoque_forn,
        l_saida_txt-estoque_forn_ltan       TO v_saida_txt_estoque_forn_ltan,
        l_saida_txt-estfat                  TO v_saida_txt_estfat,
        l_saida_txt-vol_aproduzir           TO v_saida_txt_vol_aproduzir,
        l_saida_txt-vol_aprogramar          TO v_saida_txt_vol_aprogramar,
        l_saida_txt-chmd_ag_prod_pc          TO v_saida_txt_chmd_ag_prod_pc,
        l_saida_txt-chmd_ag_prod_tn          TO v_saida_txt_chmd_ag_prod_tn,
        l_saida_txt-estoque1dia             TO v_saida_txt_estoque1dia,
        l_saida_txt-estqtran                TO v_saida_txt_estqtran,
        l_saida_txt-qtd_mp_che              TO v_saida_txt_qtd_mp_che,
        l_saida_txt-periodo_1pc             TO v_saida_txt_periodo_1pc,
        l_saida_txt-periodo_2pc             TO v_saida_txt_periodo_2pc,
        l_saida_txt-periodo_3pc             TO v_saida_txt_periodo_3pc,
        l_saida_txt-periodo_4pc             TO v_saida_txt_periodo_4pc,
        l_saida_txt-periodo_5pc             TO v_saida_txt_periodo_5pc,
        l_saida_txt-aproduzir_pec           TO v_saida_txt_aproduzir_pec,
        l_saida_txt-pendencia_pec           TO v_saida_txt_pendencia_pec,
        l_saida_txt-ov_peca                 TO v_saida_txt_ov_peca,
        l_saida_txt-planejadas_pc           TO v_saida_txt_planejadas_pc,
        l_saida_txt-volume_rem_pc           TO v_saida_txt_volume_rem_pc,
        l_saida_txt-volume_rem_1_pc         TO v_saida_txt_volume_rem_1_pc,
        l_saida_txt-requisicoes_pc          TO v_saida_txt_requisicoes_pc,
        l_saida_txt-pedidos_cen_pc          TO v_saida_txt_pedidos_cen_pc,
        l_saida_txt-op_lib_pec              TO v_saida_txt_op_lib_pec,
        l_saida_txt-op_aber_pec             TO v_saida_txt_op_aber_pec,
        l_saida_txt-expedicao_pec           TO v_saida_txt_expedicao_pec,
        l_saida_txt-expedi_cen_pc           TO v_saida_txt_expedi_cen_pc,
        l_saida_txt-exp_inter_pec           TO v_saida_txt_exp_inter_pec,
        l_saida_txt-mat_cq_pec              TO v_saida_txt_mat_cq_pec,
        l_saida_txt-faturado_pc             TO v_saida_txt_faturado_pc,
        l_saida_txt-faturado1_pc            TO v_saida_txt_faturado1_pc,
        l_saida_txt-faturado_cen_pc         TO v_saida_txt_faturado_cen_pc,
        l_saida_txt-faturado1_cen_pc        TO v_saida_txt_faturado1_cen_pc,
        l_saida_txt-prodtot_pec             TO v_saida_txt_prodtot_pec,
        l_saida_txt-analise_prev_pec        TO v_saida_txt_analise_prev_pec,
        l_saida_txt-abast                   TO v_saida_txt_abast,
        l_saida_txt-abast_pec               TO v_saida_txt_abast_pec,

        l_saida_txt-ausch                   TO v_saida_txt_ausch,
        l_saida_txt-qtd_rack                TO v_saida_txt_qtd_rack,
        l_saida_txt-prev_org_pec            TO v_saida_txt_prev_org_pec,
        l_saida_txt-prev_org_cen            TO v_saida_txt_prev_org_cen,
        l_saida_txt-prev_org_cen_pec        TO v_saida_txt_prev_org_cen_pec.

* Formatar campos de data.
*    CONCATENATE l_saida_txt-data_modif+6(2) '.' l_saida_txt-data_modif+4(2) '.' l_saida_txt-data_modif(4)
*           INTO v_saida_txt_data_modif.
*    CONCATENATE l_saida_txt-data_inicio+6(2) '.' l_saida_txt-data_inicio+4(2) '.' l_saida_txt-data_inicio(4)
*           INTO v_saida_txt_data_inicio.
    WRITE:  l_saida_txt-data_inicio   TO v_saida_txt_data_inicio,
            l_saida_txt-data_modif    TO v_saida_txt_data_modif,
            l_saida_txt-data_entre    TO v_saida_txt_data_entre,
            l_saida_txt-prazo         TO v_saida_txt_prazo,
            l_saida_txt-chmd_dt_neces TO v_saida_txt_chmd_dt_neces.

    PERFORM f_trata_negativo CHANGING v_saida_txt_peso_peca.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_minimo.
    PERFORM f_trata_negativo CHANGING v_saida_txt_est_min_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_est_min_dias.
    PERFORM f_trata_negativo CHANGING v_saida_txt_prev_org.
    PERFORM f_trata_negativo CHANGING v_saida_txt_ov_emitida.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_1.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_2.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_3.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_4.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_5.
    PERFORM f_trata_negativo CHANGING v_saida_txt_aproduzir.
    PERFORM f_trata_negativo CHANGING v_saida_txt_pendencia.
    PERFORM f_trata_negativo CHANGING v_saida_txt_ov.
    PERFORM f_trata_negativo CHANGING v_saida_txt_planejadas.
    PERFORM f_trata_negativo CHANGING v_saida_txt_volume_rem.
    PERFORM f_trata_negativo CHANGING v_saida_txt_volume_rem_1.
    PERFORM f_trata_negativo CHANGING v_saida_txt_requisicoes.
    PERFORM f_trata_negativo CHANGING v_saida_txt_pedidos.
    PERFORM f_trata_negativo CHANGING v_saida_txt_pedidos_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_pedidos_cen.
    PERFORM f_trata_negativo CHANGING v_saida_txt_op_lib.
    PERFORM f_trata_negativo CHANGING v_saida_txt_op_lib_cen.
    PERFORM f_trata_negativo CHANGING v_saida_txt_op_lib_cen_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_op_aber.
    PERFORM f_trata_negativo CHANGING v_saida_txt_op_aber_cen.
    PERFORM f_trata_negativo CHANGING v_saida_txt_op_aber_cen_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_produzir_op.
    PERFORM f_trata_negativo CHANGING v_saida_txt_expedicao.
    PERFORM f_trata_negativo CHANGING v_saida_txt_expedi_cen.
    PERFORM f_trata_negativo CHANGING v_saida_txt_exp_inter.
    PERFORM f_trata_negativo CHANGING v_saida_txt_mat_cq.
    PERFORM f_trata_negativo CHANGING v_saida_txt_faturado.
    PERFORM f_trata_negativo CHANGING v_saida_txt_faturado1.
    PERFORM f_trata_negativo CHANGING v_saida_txt_faturado_cen.
    PERFORM f_trata_negativo CHANGING v_saida_txt_faturado1_cen.
    PERFORM f_trata_negativo CHANGING v_saida_txt_prodtot.
    PERFORM f_trata_negativo CHANGING v_saida_txt_necessidade_mp.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_mp.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_mp_cen.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_consignado.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_ltan.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_ltal.
*    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_mp_cen_det.
    PERFORM f_trata_negativo CHANGING v_saida_txt_saldo_mp.
    PERFORM f_trata_negativo CHANGING v_saida_txt_analise_prev.
    PERFORM f_trata_negativo CHANGING v_saida_txt_analise.
    PERFORM f_trata_negativo CHANGING v_saida_txt_max_expe_ton.
    PERFORM f_trata_negativo CHANGING v_saida_txt_max_expe_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_forn.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque_forn_ltan.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estfat.
    PERFORM f_trata_negativo CHANGING v_saida_txt_vol_aproduzir.
    PERFORM f_trata_negativo CHANGING v_saida_txt_vol_aprogramar.
    PERFORM f_trata_negativo CHANGING v_saida_txt_chmd_ag_prod_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_chmd_ag_prod_tn.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estoque1dia.
    PERFORM f_trata_negativo CHANGING v_saida_txt_estqtran.
    PERFORM f_trata_negativo CHANGING v_saida_txt_qtd_mp_che.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_1pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_2pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_3pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_4pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_periodo_5pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_aproduzir_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_pendencia_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_ov_peca.
    PERFORM f_trata_negativo CHANGING v_saida_txt_planejadas_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_volume_rem_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_volume_rem_1_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_requisicoes_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_pedidos_cen_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_op_lib_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_op_aber_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_expedicao_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_expedi_cen_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_exp_inter_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_mat_cq_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_faturado_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_faturado1_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_faturado_cen_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_faturado1_cen_pc.
    PERFORM f_trata_negativo CHANGING v_saida_txt_prodtot_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_analise_prev_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_abast.
    PERFORM f_trata_negativo CHANGING v_saida_txt_abast_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_ausch.
    PERFORM f_trata_negativo CHANGING v_saida_txt_qtd_rack.
    PERFORM f_trata_negativo CHANGING v_saida_txt_prev_org_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_prev_org_cen.
    PERFORM f_trata_negativo CHANGING v_saida_txt_prev_org_cen_pec.
    PERFORM f_trata_negativo CHANGING v_saida_txt_ov_emitida_pc.

* Formatar campo CHAVE.
    CONCATENATE l_saida_txt-matnr l_saida_txt-werks l_saida_txt-cod_cli INTO lv_chave SEPARATED BY space.

*RDM324238 - GVCA - Begin
    CLEAR v_saida_txt_observacao.
    v_saida_txt_observacao = l_saida_txt-observacao.
*REPLACE ALL OCCURRENCES OF REGEX '\n' IN v_saida_txt_observacao WITH space.
    REPLACE ALL OCCURRENCES OF REGEX '\r' IN v_saida_txt_observacao WITH space.
*REPLACE ALL OCCURRENCES OF REGEX '\r\n' IN v_saida_txt_observacao WITH space.
*RDM324238 - GVCA - End

* Concatenar para a linha do t_texto.
    CONCATENATE
             lv_chave
        ';' l_saida_txt-matnr
        ';' v_saida_txt_observacao
        ';' l_saida_txt-werks
        ';' l_saida_txt-wrk02
        ';' l_saida_txt-descricao
        ';' l_saida_txt-lgort
        ';' l_saida_txt-cliente
        ';' l_saida_txt-cod_cli
        ';' l_saida_txt-codigo
        ';' l_saida_txt-projeto
        ';' v_saida_txt_peso_peca
        ';' v_saida_txt_ausch
        ';' l_saida_txt-ferramenta
        ';' l_saida_txt-roteiro
        ';' l_saida_txt-embalagem
        ';' v_saida_txt_qtd_rack
        ';' l_saida_txt-tamanho_lote
        ';' v_saida_txt_estoque_minimo
        ';' v_saida_txt_est_min_pec
        ';' v_saida_txt_est_min_dias
        ';' v_saida_txt_prev_org
        ';' v_saida_txt_prev_org_pec
        ';' v_saida_txt_prev_org_cen
        ';' v_saida_txt_prev_org_cen_pec
        ';' v_saida_txt_ov_emitida
        ';' v_saida_txt_ov_emitida_pc
        ';' v_saida_txt_periodo_1
        ';' v_saida_txt_periodo_1pc
        ';' v_saida_txt_periodo_2
        ';' v_saida_txt_periodo_2pc
        ';' v_saida_txt_periodo_3
        ';' v_saida_txt_periodo_3pc
        ';' v_saida_txt_periodo_4
        ';' v_saida_txt_periodo_4pc
        ';' v_saida_txt_periodo_5
        ';' v_saida_txt_periodo_5pc
        ';' v_saida_txt_aproduzir
        ';' v_saida_txt_aproduzir_pec
        ';' v_saida_txt_pendencia
        ';' v_saida_txt_pendencia_pec
        ';' v_saida_txt_ov
        ';' v_saida_txt_ov_peca
        ';' v_saida_txt_planejadas
        ';' v_saida_txt_planejadas_pc
        ';' v_saida_txt_data_inicio
        ';' v_saida_txt_volume_rem
        ';' v_saida_txt_volume_rem_pc
        ';' v_saida_txt_volume_rem_1
        ';' v_saida_txt_volume_rem_1_pc
        ';' v_saida_txt_requisicoes
        ';' v_saida_txt_requisicoes_pc
        ';' v_saida_txt_pedidos
        ';' l_saida_txt-fornecedor
        ';' v_saida_txt_pedidos_pc
        ';' v_saida_txt_pedidos_cen
        ';' v_saida_txt_pedidos_cen_pc
        ';' v_saida_txt_op_lib
        ';' v_saida_txt_op_lib_pec
        ';' v_saida_txt_op_lib_cen
        ';' v_saida_txt_op_lib_cen_pec
        ';' v_saida_txt_op_aber
        ';' v_saida_txt_op_aber_pec
        ';' v_saida_txt_op_aber_cen
        ';' v_saida_txt_op_aber_cen_pec
        ';' v_saida_txt_produzir_op
        ';' v_saida_txt_prazo
        ';' v_saida_txt_expedicao
        ';' v_saida_txt_expedicao_pec
        ';' v_saida_txt_expedi_cen
        ';' v_saida_txt_expedi_cen_pc
        ';' v_saida_txt_exp_inter
        ';' v_saida_txt_exp_inter_pec
        ';' l_saida_txt-expedi_cen_det
        ';' v_saida_txt_mat_cq
        ';' v_saida_txt_mat_cq_pec
        ';' v_saida_txt_faturado
        ';' v_saida_txt_faturado1
        ';' v_saida_txt_faturado_pc
        ';' v_saida_txt_faturado1_pc
        ';' v_saida_txt_faturado_cen
        ';' v_saida_txt_faturado_cen_pc
        ';' v_saida_txt_faturado1_cen
        ';' v_saida_txt_faturado1_cen_pc
        ';' v_saida_txt_prodtot
        ';' v_saida_txt_prodtot_pec
        ';' v_saida_txt_necessidade_mp
        ';' l_saida_txt-materia_prima
        ';' l_saida_txt-desc_mp
        ';' v_saida_txt_estoque_mp
        ';' v_saida_txt_estoque_mp_cen
        ';' v_saida_txt_estoque_consignado
        ';' v_saida_txt_estoque_ltan
        ';' v_saida_txt_estoque_ltal
        ';' v_saida_txt_estoque_mp_cen_det
        ';' v_saida_txt_saldo_mp
        ';' v_saida_txt_analise_prev
        ';' v_saida_txt_analise_prev_pec
        ';' v_saida_txt_analise
        ';' v_saida_txt_abast
        ';' v_saida_txt_abast_pec
        ';' v_saida_txt_max_expe_ton
        ';' v_saida_txt_max_expe_pc
        ';' v_saida_txt_data_modif
        ';' l_saida_txt-meins
        ';' l_saida_txt-h_material
        ';' l_saida_txt-l_entrega
        ';' l_saida_txt-ativo
        ';' l_saida_txt-producao
        ';' v_saida_txt_estoque_forn
        ';' v_saida_txt_estoque_forn_ltan
        ';' v_saida_txt_estfat
        ';' v_saida_txt_vol_aproduzir
        ';' v_saida_txt_vol_aprogramar
        ';' v_saida_txt_chmd_ag_prod_pc
        ';' v_saida_txt_chmd_ag_prod_tn
        ';' v_saida_txt_chmd_dt_neces
        ';' v_saida_txt_estoque1dia
        ';' l_saida_txt-cobfat_exp_org
        ';' l_saida_txt-cob_exp_org
        ';' l_saida_txt-cob_fat_org
        ';' l_saida_txt-cob_op_total_dem_org
        ';' l_saida_txt-cob_mp_forne_dem_org
        ';' l_saida_txt-cob_ped_comp_org
        ';' l_saida_txt-cob_exp_60
        ';' l_saida_txt-cobfat_exp_60
        ';' l_saida_txt-cob_stat_exp_60
        ';' l_saida_txt-status_cobert_exp_org
        ';' l_saida_txt-vol_mensal_atendido
        ';' l_saida_txt-soma_mps
        ';' v_saida_txt_estqtran
        ';' l_saida_txt-status_pcp
        ';' v_saida_txt_qtd_mp_che
        ';' v_saida_txt_data_entre

    INTO l_texto_str.
    APPEND l_texto_str TO t_texto_str.

    CLEAR l_texto_str.

  ENDLOOP.
*  DATA l_data TYPE REF TO data.
*
*  LOOP AT t_saida ASSIGNING FIELD-SYMBOL(<fs_saida>).
*    CLEAR l_texto_str.
*
**    LOOP AT lt_comp_tab ASSIGNING <fs_component>.
**      lv_tabix = sy-tabix.
**
**      READ TABLE gt_fieldcat ASSIGNING <fs_fcat> WITH KEY fieldname = <fs_component>-name.
*
*    LOOP AT gt_fieldcat ASSIGNING <fs_fcat>.
*      lv_tabix = sy-tabix.
*
*      READ TABLE lt_comp_tab ASSIGNING <fs_component> WITH KEY name = <fs_fcat>-fieldname.
*
*      CHECK sy-subrc = 0 AND <fs_fcat>-fieldname NOT IN lr_ignore_fields.
*
*      ASSIGN COMPONENT <fs_component>-name OF STRUCTURE <fs_saida> TO <fs_value>.
*
*      IF <fs_component>-name = 'DATA_INICIO' OR
*         <fs_component>-name = 'DATA_MODIF' OR
*         <fs_component>-name = 'DATA_ENTRE' OR
*         <fs_component>-name = 'PRAZO'.
*
*        DATA(lv_var_name) = |V_SAIDA_TXT_{ <fs_component>-name }|.
*
*        ASSIGN (lv_var_name) TO <fs_variable>.
*
*        CHECK <fs_variable> IS ASSIGNED.
*
*        WRITE <fs_value> TO <fs_variable>.
*
*        <fs_value> = <fs_variable>.
*      ELSEIF <fs_component>-name = 'OBSERVACAO'.
*        CLEAR v_saida_txt_observacao.
*        v_saida_txt_observacao = <fs_saida>-observacao.
*
*        REPLACE ALL OCCURRENCES OF REGEX '\r' IN v_saida_txt_observacao WITH space.
*
*        ASSIGN v_saida_txt_observacao TO <fs_value>.
*      ELSE.
*        lv_var_name = |V_SAIDA_TXT_{ <fs_component>-name }|.
*
*        ASSIGN (lv_var_name) TO <fs_variable>.
*
*        IF <fs_variable> IS ASSIGNED.
*          MOVE <fs_value> TO <fs_variable>.
*          <fs_value> = <fs_variable>.
*        ENDIF.
*      ENDIF.
*
*      IF lv_tabix = 1.
*        l_texto_str-field = <fs_value>.
*      ELSE.
*        l_texto_str-field = |{ l_texto_str-field };{ <fs_value> }|.
*      ENDIF.
*    ENDLOOP.
*
*    APPEND l_texto_str TO t_texto_str.
*  ENDLOOP.

ENDFORM.                    " f_preparar_impressao_csv
***Inserção - LG - RDM 334177 - 15/04/2026 - Início - FORM F_PRE_CALCULA_CAMPOS_EF
*&---------------------------------------------------------------------*
*&      Form  F_PRE_CALCULA_CAMPOS_EF
*&---------------------------------------------------------------------*
FORM f_pre_calcula_campos_ef.

  TYPES: BEGIN OF ty_chave_saida,
           matnr   TYPE mara-matnr,
           werks   TYPE crhd-werks,
           cod_cli TYPE kna1-kunnr,
         END OF ty_chave_saida,
         BEGIN OF ty_chave_ped,
           matnr TYPE mara-matnr,
           werks TYPE crhd-werks,
         END OF ty_chave_ped,
         BEGIN OF ty_chave_mat_werks,
           matnr TYPE mara-matnr,
           werks TYPE crhd-werks,
         END OF ty_chave_mat_werks,
         BEGIN OF ty_popa_aux,
           matnr  TYPE mara-matnr,
*           werks  TYPE crhd-werks,
           znumpp TYPE zrnppt_popa-znumpp,
         END OF ty_popa_aux,
         BEGIN OF ty_aufk_aux_ef,
           aufnr TYPE aufk-aufnr,
           objnr TYPE aufk-objnr,
         END OF ty_aufk_aux_ef,
         BEGIN OF ty_jest_aux,
           objnr TYPE jest-objnr,
           stat  TYPE jest-stat,
           inact TYPE jest-inact,
         END OF ty_jest_aux,
         BEGIN OF ty_pconf_aux,
           znumpp TYPE zrnppt_pconfit-znumpp,
         END OF ty_pconf_aux,
         BEGIN OF ty_ekpo_aux_ef,
           matnr TYPE ekpo-matnr,
           werks TYPE ekpo-werks,
           ebeln TYPE ekpo-ebeln,
           ebelp TYPE ekpo-ebelp,
         END OF ty_ekpo_aux_ef,
         BEGIN OF ty_pedido_aux_ef,
           matnr TYPE ekpo-matnr,
           werks TYPE ekpo-werks,
           ebeln TYPE ekpo-ebeln,
         END OF ty_pedido_aux_ef,
         BEGIN OF ty_ekbe_aux_ef,
           ebeln TYPE ekbe-ebeln,
           ebelp TYPE ekbe-ebelp,
         END OF ty_ekbe_aux_ef.

  DATA: lt_chaves       TYPE SORTED TABLE OF ty_chave_saida WITH UNIQUE KEY matnr werks cod_cli,
        ls_chave        TYPE ty_chave_saida,
        lt_chaves_ped   TYPE SORTED TABLE OF ty_chave_ped WITH UNIQUE KEY matnr werks,
        ls_chave_ped    TYPE ty_chave_ped,
        lt_chaves_mw    TYPE SORTED TABLE OF ty_chave_mat_werks WITH UNIQUE KEY matnr werks,
        ls_chave_mw     TYPE ty_chave_mat_werks,
        lt_popa         TYPE TABLE OF ty_popa_aux,
        ls_popa         TYPE ty_popa_aux,
        lt_aufk         TYPE TABLE OF ty_aufk_aux_ef,
        ls_aufk         TYPE ty_aufk_aux_ef,
        lt_jest         TYPE TABLE OF ty_jest_aux,
        ls_jest         TYPE ty_jest_aux,
        lt_pconf        TYPE TABLE OF ty_pconf_aux,
        ls_pconf        TYPE ty_pconf_aux,
        lt_ekpo         TYPE TABLE OF ty_ekpo_aux_ef,
        ls_ekpo         TYPE ty_ekpo_aux_ef,
        lt_pedido       TYPE SORTED TABLE OF ty_pedido_aux_ef WITH UNIQUE KEY matnr werks ebeln,
        ls_pedido       TYPE ty_pedido_aux_ef,
        lt_ekbe         TYPE TABLE OF ty_ekbe_aux_ef,
        ls_ekbe         TYPE ty_ekbe_aux_ef,
        lt_vbeln        TYPE SORTED TABLE OF vbak-vbeln WITH UNIQUE KEY table_line,
        lv_vbeln        TYPE vbak-vbeln,
        lv_ref_date     TYPE sy-datum,
        lv_yymm_m1      TYPE n LENGTH 6,
        lv_yymm_m2      TYPE n LENGTH 6,
        lv_yymm_m3      TYPE n LENGTH 6,
        lv_has_aber_lib TYPE c,
        lv_has_conf     TYPE c,
        lv_count        TYPE i.

  IF p_corte IS INITIAL.
    RETURN.
  ENDIF.

  IF gv_ef_precalc = 'X'.
    RETURN.
  ENDIF.

  CLEAR: gt_ef_fat_hist_sum, gt_ef_op_sem_mov, gt_ef_ped_sem_mov, gt_ef_vbak_remessa.

  LOOP AT t_saida INTO wa_saida.
    ls_chave-matnr   = wa_saida-matnr.
    ls_chave-werks   = wa_saida-werks.
    ls_chave-cod_cli = wa_saida-cod_cli.
    INSERT ls_chave INTO TABLE lt_chaves.

    ls_chave_mw-matnr = wa_saida-matnr.
    ls_chave_mw-werks = wa_saida-werks.
    INSERT ls_chave_mw INTO TABLE lt_chaves_mw.

    IF wa_saida-pedidos > 0.
      ls_chave_ped-matnr = wa_saida-matnr.
      ls_chave_ped-werks = wa_saida-werks.
      INSERT ls_chave_ped INTO TABLE lt_chaves_ped.
    ENDIF.
  ENDLOOP.

  IF lt_chaves[] IS INITIAL.
    gv_ef_precalc = 'X'.
    RETURN.
  ENDIF.

  lv_ref_date = p_period-low.
  IF lv_ref_date IS INITIAL.
    lv_ref_date = sy-datum.
  ENDIF.

  CALL FUNCTION 'MONTH_PLUS_DETERMINE'
    EXPORTING
      months  = -1
      olddate = lv_ref_date
    IMPORTING
      newdate = lv_ref_date.
  lv_yymm_m1 = lv_ref_date(6).

  CALL FUNCTION 'MONTH_PLUS_DETERMINE'
    EXPORTING
      months  = -1
      olddate = lv_ref_date
    IMPORTING
      newdate = lv_ref_date.
  lv_yymm_m2 = lv_ref_date(6).

  CALL FUNCTION 'MONTH_PLUS_DETERMINE'
    EXPORTING
      months  = -1
      olddate = lv_ref_date
    IMPORTING
      newdate = lv_ref_date.
  lv_yymm_m3 = lv_ref_date(6).

  lv_ref_date = |{ lv_yymm_m1 }01|.

  CALL FUNCTION 'LAST_DAY_OF_MONTHS'
    EXPORTING
      day_in            = lv_ref_date
    IMPORTING
      last_day_of_month = lv_ref_date
    EXCEPTIONS
      day_in_no_date    = 1
      OTHERS            = 2.

  LOOP AT t_vbrp.
    IF t_vbrp-aubel IS NOT INITIAL.
      lv_vbeln = t_vbrp-aubel.
      INSERT lv_vbeln INTO TABLE lt_vbeln.
    ENDIF.
  ENDLOOP.

  IF lt_vbeln[] IS NOT INITIAL.
    SELECT vbeln vdatu
      INTO TABLE gt_ef_vbak_remessa
      FROM vbak
      FOR ALL ENTRIES IN lt_vbeln
      WHERE vbeln = lt_vbeln-table_line.
  ENDIF.

  DATA lr_periodo TYPE RANGE OF sy-datum.
  lr_periodo = VALUE #( sign = 'I' option = 'BT' ( low = |{ lv_yymm_m3(6) }01| high = |{ lv_ref_date }| ) ).

  IF t_vbrp[] IS NOT INITIAL.

    SELECT b~matnr, b~werks, b~vbeln, b~posnr, b~aubel, a~kunag, a~fkdat, c~vbeln AS venda, c~vdatu, b~fklmg
      FROM vbrk AS a
      INNER JOIN vbrp AS b
        ON a~vbeln = b~vbeln
      INNER JOIN vbak AS c
        ON b~aubel = c~vbeln
      FOR ALL ENTRIES IN @t_vbrp
*        WHERE b~werks = @t_vbrp-werks
*        AND b~matnr   = @t_vbrp-matnr
      WHERE b~matnr   = @t_vbrp-matnr
        AND a~kunag   = @t_vbrp-kunag
        AND a~fkdat   IN @lr_periodo
        AND c~vdatu   IN @lr_periodo
        AND b~pstyv   <> 'ZCFO'
      INTO TABLE @DATA(lt_vbrp).

    IF sy-subrc EQ 0.

      LOOP AT lt_vbrp INTO DATA(ls_vbrp).
        CLEAR: gs_ef_fat_hist_sum.
        gs_ef_fat_hist_sum-matnr = ls_vbrp-matnr.
        gs_ef_fat_hist_sum-werks = ls_vbrp-werks.
        gs_ef_fat_hist_sum-kunag = ls_vbrp-kunag.

        IF ls_vbrp-vdatu(6) = lv_yymm_m1.
          gs_ef_fat_hist_sum-m1 = ls_vbrp-fklmg.
        ELSEIF ls_vbrp-vdatu(6) = lv_yymm_m2.
          gs_ef_fat_hist_sum-m2 = ls_vbrp-fklmg.
        ELSEIF ls_vbrp-vdatu(6) = lv_yymm_m3.
          gs_ef_fat_hist_sum-m3 = ls_vbrp-fklmg.
        ENDIF.

        COLLECT gs_ef_fat_hist_sum INTO gt_ef_fat_hist_sum.

      ENDLOOP.

    ENDIF.

  ENDIF.



*  LOOP AT t_vbrp.
*    READ TABLE lt_chaves INTO ls_chave WITH KEY matnr = t_vbrp-matnr
*                                               werks = t_vbrp-werks
*                                               cod_cli = t_vbrp-kunag BINARY SEARCH.
*    IF sy-subrc <> 0.
*      CONTINUE.
*    ENDIF.
*
*    lv_ref_date = t_vbrp-erdat.
*    IF t_vbrp-aubel IS NOT INITIAL.
*      READ TABLE gt_ef_vbak_remessa INTO gs_ef_vbak_remessa WITH KEY vbeln = t_vbrp-aubel BINARY SEARCH.
*      IF sy-subrc = 0 AND gs_ef_vbak_remessa-vdatu IS NOT INITIAL.
*        IF gs_ef_vbak_remessa-vdatu(6) <> t_vbrp-erdat(6).
*          lv_ref_date = gs_ef_vbak_remessa-vdatu.
*        ENDIF.
*      ENDIF.
*    ENDIF.
*
*    READ TABLE gt_ef_fat_hist_sum INTO gs_ef_fat_hist_sum WITH KEY matnr = t_vbrp-matnr
*                                                                    werks = t_vbrp-werks
*                                                                    kunag = t_vbrp-kunag.
*    IF sy-subrc EQ 0.
*      DATA(lv_tabix) = sy-tabix.
*    ELSE.
*      CLEAR: gs_ef_fat_hist_sum, lv_tabix.
*      gs_ef_fat_hist_sum-matnr = t_vbrp-matnr.
*      gs_ef_fat_hist_sum-werks = t_vbrp-werks.
*      gs_ef_fat_hist_sum-kunag = t_vbrp-kunag.
*    ENDIF.
*
*    IF lv_ref_date(6) = lv_yymm_m3.
*      gs_ef_fat_hist_sum-m3 = gs_ef_fat_hist_sum-m3 + t_vbrp-ntgew.
*    ELSEIF lv_ref_date(6) = lv_yymm_m2.
*      gs_ef_fat_hist_sum-m2 = gs_ef_fat_hist_sum-m2 + t_vbrp-ntgew.
*    ELSEIF lv_ref_date(6) = lv_yymm_m1.
*      gs_ef_fat_hist_sum-m1 = gs_ef_fat_hist_sum-m1 + t_vbrp-ntgew.
*    ENDIF.
*
*    IF lv_tabix IS NOT INITIAL.
*      MODIFY TABLE gt_ef_fat_hist_sum FROM gs_ef_fat_hist_sum.
*    ELSE.
*      INSERT gs_ef_fat_hist_sum INTO TABLE gt_ef_fat_hist_sum .
*    ENDIF.
*  ENDLOOP.

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

  IF lt_chaves_ped[] IS NOT INITIAL.

    SELECT p~matnr, p~werks, p~ebeln, p~ebelp
      INTO TABLE @lt_ekpo
      FROM ekpo AS p
      INNER JOIN ekko AS k
      ON p~ebeln = k~ebeln
      FOR ALL ENTRIES IN @lt_chaves_ped
      WHERE p~matnr = @lt_chaves_ped-matnr
        AND p~werks = @lt_chaves_ped-werks
        AND p~elikz NE @abap_true  "= @abap_false
        AND p~loekz EQ @abap_false
        AND p~pstyp EQ '3'
        AND p~ebeln <> ''
        AND k~bstyp EQ 'F'. "Pedido

    IF sy-subrc EQ 0.

      SORT lt_ekpo BY matnr werks ebeln ebelp.

      SELECT DISTINCT ebeln, ebelp
        INTO TABLE @DATA(lt_mseg)
        FROM mseg
        FOR ALL ENTRIES IN @lt_ekpo
        WHERE ebeln EQ @lt_ekpo-ebeln
          AND ebelp EQ @lt_ekpo-ebelp.

      IF sy-subrc EQ 0.
        SORT lt_mseg BY ebeln ASCENDING
                        ebelp ASCENDING.
      ENDIF.

      SELECT DISTINCT ebeln, ebelp
        INTO TABLE @DATA(lt_ekbe_aux)
        FROM ekbe
        FOR ALL ENTRIES IN @lt_ekpo
        WHERE ebeln EQ @lt_ekpo-ebeln
          AND ebelp EQ @lt_ekpo-ebelp
          AND bwart IN ( '101', '543' )
          AND elikz NE @abap_true.

      IF sy-subrc EQ 0.
        SORT lt_ekbe_aux BY ebeln ASCENDING
                            ebelp ASCENDING.
      ENDIF.

      LOOP AT lt_ekpo INTO ls_ekpo.

        READ TABLE lt_mseg WITH KEY ebeln = ls_ekpo-ebeln
                                    ebelp = ls_ekpo-ebelp
                                    TRANSPORTING NO FIELDS
                                    BINARY SEARCH.
        IF sy-subrc EQ 0.
          CONTINUE.
        ENDIF.

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

    ENDIF.

*    IF lt_ekpo[] IS NOT INITIAL.
*      SELECT ebeln ebelp
*        INTO TABLE lt_ekbe
*        FROM ekbe
*        FOR ALL ENTRIES IN lt_ekpo
*        WHERE ebeln = lt_ekpo-ebeln
*          AND ebelp = lt_ekpo-ebelp
*          AND elikz = abap_false.
**          AND bewtp = 'E'.
*
*      SORT lt_ekbe BY ebeln ebelp.
*      DELETE ADJACENT DUPLICATES FROM lt_ekbe COMPARING ebeln ebelp.
*
*      LOOP AT lt_ekpo INTO ls_ekpo.
*        ls_pedido-matnr = ls_ekpo-matnr.
*        ls_pedido-werks = ls_ekpo-werks.
*        ls_pedido-ebeln = ls_ekpo-ebeln.
*        INSERT ls_pedido INTO TABLE lt_pedido.
*      ENDLOOP.
*    ENDIF.

*    LOOP AT lt_pedido INTO ls_pedido.
*      lv_count = 0.
*      LOOP AT lt_ekpo INTO ls_ekpo WHERE matnr = ls_pedido-matnr
*                                     AND werks = ls_pedido-werks
*                                     AND ebeln = ls_pedido-ebeln.
*        READ TABLE lt_ekbe INTO ls_ekbe WITH KEY ebeln = ls_ekpo-ebeln
*                                                 ebelp = ls_ekpo-ebelp BINARY SEARCH.
*        IF sy-subrc = 0.
**          lv_count = 1.
*          READ TABLE gt_ef_ped_sem_mov INTO gs_ef_ped_sem_mov WITH KEY matnr = ls_pedido-matnr
*                                                                        werks = ls_pedido-werks.
*          IF sy-subrc EQ 0.
*            lv_tabix = sy-tabix.
*          ELSE.
*            CLEAR: gs_ef_ped_sem_mov, lv_tabix.
*            gs_ef_ped_sem_mov-matnr = ls_pedido-matnr.
*            gs_ef_ped_sem_mov-werks = ls_pedido-werks.
*          ENDIF.
*          gs_ef_ped_sem_mov-qtd = gs_ef_ped_sem_mov-qtd + 1.
*          IF lv_tabix IS NOT INITIAL.
*            MODIFY TABLE gt_ef_ped_sem_mov FROM gs_ef_ped_sem_mov.
*          ELSE.
*            INSERT gs_ef_ped_sem_mov INTO TABLE gt_ef_ped_sem_mov.
*          ENDIF.
*          EXIT.
*        ENDIF.
*      ENDLOOP.
*
**      IF lv_count = 0.
**        READ TABLE gt_ef_ped_sem_mov INTO gs_ef_ped_sem_mov WITH KEY matnr = ls_pedido-matnr
**                                                                      werks = ls_pedido-werks.
**        IF sy-subrc EQ 0.
**          lv_tabix = sy-tabix.
**        ELSE.
**          CLEAR: gs_ef_ped_sem_mov, lv_tabix.
**          gs_ef_ped_sem_mov-matnr = ls_pedido-matnr.
**          gs_ef_ped_sem_mov-werks = ls_pedido-werks.
**        ENDIF.
**        gs_ef_ped_sem_mov-qtd = gs_ef_ped_sem_mov-qtd + 1.
**        IF lv_tabix IS NOT INITIAL.
**          MODIFY TABLE gt_ef_ped_sem_mov FROM gs_ef_ped_sem_mov.
**        ELSE.
**          INSERT gs_ef_ped_sem_mov INTO TABLE gt_ef_ped_sem_mov.
**        ENDIF.
**      ENDIF.
*    ENDLOOP.
  ENDIF.

  gv_ef_precalc = 'X'.

ENDFORM.

***Inserção - LG - RDM 334177 - 15/04/2026 - Fim - FORM F_PRE_CALCULA_CAMPOS_EF

***Inserção - LG - RDM 334177 - 15/04/2026 - Início - FORM F_APLICA_CAMPOS_EF
*&---------------------------------------------------------------------*
*&      Form  F_APLICA_CAMPOS_EF
*&---------------------------------------------------------------------*
FORM f_aplica_campos_ef CHANGING p_saida LIKE t_saida.

  FIELD-SYMBOLS: <fs_any> TYPE any.

  DATA: lv_prev_org     TYPE p DECIMALS 3,
        lv_prev_tn1     TYPE p DECIMALS 3,
        lv_prev_tn2     TYPE p DECIMALS 3,
        lv_ped_comp     TYPE p DECIMALS 3,
        lv_ped_comp_cen TYPE p DECIMALS 3,
        lv_op_aber      TYPE p DECIMALS 3,
        lv_op_aber_cen  TYPE p DECIMALS 3,
        lv_exped        TYPE p DECIMALS 3,
        lv_exped_cen    TYPE p DECIMALS 3,
        lv_fat          TYPE p DECIMALS 3,
        lv_fat_cen      TYPE p DECIMALS 3,
        lv_delta        TYPE p DECIMALS 3,
        lv_cod_cli      TYPE kna1-kunnr.

  IF p_corte IS INITIAL.
    RETURN.
  ENDIF.

  CLEAR: lv_prev_org, lv_prev_tn1, lv_prev_tn2, lv_ped_comp, lv_ped_comp_cen,
         lv_op_aber, lv_op_aber_cen, lv_exped, lv_exped_cen, lv_fat, lv_fat_cen,
         lv_delta, lv_cod_cli.

  ASSIGN COMPONENT 'PREV_ORG' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_prev_org = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'PERIODO_1' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_prev_tn1 = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'PERIODO_2' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_prev_tn2 = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'PEDIDOS' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_ped_comp = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'PEDIDOS_CEN' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_ped_comp_cen = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'OP_ABER' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_op_aber = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'OP_ABER_CEN' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_op_aber_cen = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'EXPEDICAO' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_exped = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'EXPEDI_CEN' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_exped_cen = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'FATURADO1' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_fat = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'FATURADO1_CEN' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_fat_cen = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'COD_CLI' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    lv_cod_cli = <fs_any>.
  ENDIF.

  ASSIGN COMPONENT 'ZZDEMADTOT' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    <fs_any> = lv_prev_org + lv_prev_tn1 + lv_prev_tn2.
  ENDIF.

  ASSIGN COMPONENT 'ZZPRODTOT' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    <fs_any> = lv_ped_comp + lv_ped_comp_cen + lv_op_aber + lv_op_aber_cen.
  ENDIF.

  ASSIGN COMPONENT 'ZZEXPDTOT' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    <fs_any> = lv_exped + lv_exped_cen.
  ENDIF.

  ASSIGN COMPONENT 'ZZFATTOT' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    <fs_any> = lv_fat + lv_fat_cen.
  ENDIF.

  ASSIGN COMPONENT 'ZZPRODEXPFAT' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    <fs_any> = ( lv_ped_comp + lv_ped_comp_cen + lv_op_aber + lv_op_aber_cen ) +
               ( lv_exped + lv_exped_cen ) +
               ( lv_fat + lv_fat_cen ).
  ENDIF.

  lv_delta = ( ( lv_ped_comp + lv_ped_comp_cen + lv_op_aber + lv_op_aber_cen ) +
               ( lv_exped + lv_exped_cen ) +
               ( lv_fat + lv_fat_cen ) ) -
             ( lv_prev_org + lv_prev_tn1 + lv_prev_tn2 ).

  IF lv_delta < 0.
    lv_delta = 0.
  ENDIF.

  ASSIGN COMPONENT 'ZZDELTA' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    <fs_any> = lv_delta.
  ENDIF.

  ASSIGN COMPONENT 'ZZACAO' OF STRUCTURE p_saida TO <fs_any>.
  IF sy-subrc = 0.
    IF lv_delta > 0 AND ( lv_ped_comp + lv_ped_comp_cen + lv_op_aber + lv_op_aber_cen ) > 0.
      <fs_any> = 'PCP Verificar saldo em produção: Item com demanda reduzida e/ou cancelada'.
    ELSE.
      CLEAR <fs_any>.
    ENDIF.
  ENDIF.

  READ TABLE gt_ef_op_sem_mov INTO gs_ef_op_sem_mov WITH KEY matnr = p_saida-matnr
                                                              werks = p_saida-werks.
  IF sy-subrc EQ 0.
    ASSIGN COMPONENT 'ZZOPSEMMOV' OF STRUCTURE p_saida TO <fs_any>.
    IF sy-subrc = 0.
      <fs_any> = gs_ef_op_sem_mov-qtd.
    ENDIF.
  ENDIF.

  READ TABLE gt_ef_ped_sem_mov INTO gs_ef_ped_sem_mov WITH KEY matnr = p_saida-matnr
                                                                werks = p_saida-werks.
  IF sy-subrc EQ 0.
    ASSIGN COMPONENT 'ZZPEDSEMMOV' OF STRUCTURE p_saida TO <fs_any>.
    IF sy-subrc = 0.
      <fs_any> = gs_ef_ped_sem_mov-qtd.
    ENDIF.
  ENDIF.

  READ TABLE gt_ef_fat_hist_sum INTO gs_ef_fat_hist_sum WITH KEY matnr = p_saida-matnr
                                                                  werks = p_saida-werks
                                                                  kunag = lv_cod_cli.
  IF sy-subrc EQ 0.
    ASSIGN COMPONENT 'ZZFATORG3' OF STRUCTURE p_saida TO <fs_any>.
    IF sy-subrc = 0.
      <fs_any> = gs_ef_fat_hist_sum-m3.
    ENDIF.

    ASSIGN COMPONENT 'ZZFATORG2' OF STRUCTURE p_saida TO <fs_any>.
    IF sy-subrc = 0.
      <fs_any> = gs_ef_fat_hist_sum-m2.
    ENDIF.

    ASSIGN COMPONENT 'ZZFATORG1' OF STRUCTURE p_saida TO <fs_any>.
    IF sy-subrc = 0.
      <fs_any> = gs_ef_fat_hist_sum-m1.
    ENDIF.

    ASSIGN COMPONENT 'ZZFATTOTHIST' OF STRUCTURE p_saida TO <fs_any>.
    IF sy-subrc = 0.
      <fs_any> = gs_ef_fat_hist_sum-m3 + gs_ef_fat_hist_sum-m2 + gs_ef_fat_hist_sum-m1.
    ENDIF.

  ENDIF.

ENDFORM.

***Inserção - LG - RDM 334177 - 15/04/2026 - Fim - FORM F_APLICA_CAMPOS_EF

*&---------------------------------------------------------------------*
*&      Form  F_TRATA_NEGATIVO
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      <--P_VALUE  text
*----------------------------------------------------------------------*
FORM f_trata_negativo  CHANGING p_value.

  DATA lv_float TYPE f.
  DATA lv_str TYPE string.

  lv_str = p_value.
  CONDENSE lv_str NO-GAPS.

  REPLACE ALL OCCURRENCES OF '.' IN lv_str WITH ''.
  REPLACE ALL OCCURRENCES OF ',' IN lv_str WITH '.'.

  lv_float = lv_str.

  IF lv_float < 0.
    CALL FUNCTION 'CLOI_PUT_SIGN_IN_FRONT'
      CHANGING
        value = p_value.
  ENDIF.

  " Teste 1234


ENDFORM.
*RDM324238 - End


*&---------------------------------------------------------------------*
*&  RDM 334177 - Callbacks popup Pedido Sem Movimentação
*&---------------------------------------------------------------------*

*----------------------------------------------------------------------*
* FORM F_PED_DET_PF_STATUS: define botões na toolbar do grid de pedidos
*----------------------------------------------------------------------*
FORM f_ped_det_pf_status USING extab TYPE slis_t_extab.
  SET PF-STATUS 'ZPED_DET' EXCLUDING extab.
ENDFORM.

*----------------------------------------------------------------------*
* FORM F_PED_DET_USER_COMMAND: trata ações do grid de pedidos
*----------------------------------------------------------------------*
FORM f_ped_det_user_command USING r_ucomm    TYPE sy-ucomm
                                  rs_selfield TYPE slis_selfield.
  DATA: lv_ebeln TYPE ekpo-ebeln.

  CASE r_ucomm.
    WHEN '&IC1'.  "Hotspot / duplo-clique no EBELN → ME23N
      READ TABLE gt_ef_ped_det INTO gs_ef_ped_det INDEX rs_selfield-tabindex.
      IF sy-subrc = 0 AND rs_selfield-fieldname = 'EBELN'.
        SET PARAMETER ID 'BES' FIELD gs_ef_ped_det-ebeln.
        CALL TRANSACTION 'ME23N' AND SKIP FIRST SCREEN.
      ENDIF.

    WHEN 'ZME52N'.  "Botão → ME52N (Solicitação de Compra)
      READ TABLE gt_ef_ped_det INTO gs_ef_ped_det INDEX rs_selfield-tabindex.
      IF sy-subrc = 0 AND gs_ef_ped_det-banfn IS NOT INITIAL.
        SET PARAMETER ID 'BAN' FIELD gs_ef_ped_det-banfn.
        CALL TRANSACTION 'ME52N' AND SKIP FIRST SCREEN.
      ELSE.
        MESSAGE 'Selecione um pedido com ReqC para acessar ME52N' TYPE 'I'.
      ENDIF.

    WHEN 'ZME22N'.  "Botão → ME22N (Modificar Pedido)
      READ TABLE gt_ef_ped_det INTO gs_ef_ped_det INDEX rs_selfield-tabindex.
      IF sy-subrc = 0 AND gs_ef_ped_det-ebeln IS NOT INITIAL.
        SET PARAMETER ID 'BES' FIELD gs_ef_ped_det-ebeln.
        CALL TRANSACTION 'ME22N' AND SKIP FIRST SCREEN.
      ELSE.
        MESSAGE 'Selecione um pedido para acessar ME22N' TYPE 'I'.
      ENDIF.

  ENDCASE.
ENDFORM.

*&---------------------------------------------------------------------*
*&  RDM 334177 - Callbacks popup OP Sem Movimentação
*&---------------------------------------------------------------------*

*----------------------------------------------------------------------*
* FORM F_OP_DET_PF_STATUS: define botões na toolbar do grid de OPs
*----------------------------------------------------------------------*
FORM f_op_det_pf_status USING extab TYPE slis_t_extab.
  SET PF-STATUS 'ZOP_DET' EXCLUDING extab.
ENDFORM.

*----------------------------------------------------------------------*
* FORM F_OP_DET_USER_COMMAND: trata ações do grid de OPs
*----------------------------------------------------------------------*
FORM f_op_det_user_command USING r_ucomm    TYPE sy-ucomm
                                 rs_selfield TYPE slis_selfield.

  CASE r_ucomm.
    WHEN '&IC1'.  "Hotspot / duplo-clique no AUFNR → CO03
      READ TABLE gt_ef_op_det INTO gs_ef_op_det INDEX rs_selfield-tabindex.
      IF sy-subrc = 0 AND rs_selfield-fieldname = 'AUFNR'.
        SET PARAMETER ID 'ANR' FIELD gs_ef_op_det-aufnr.
        CALL TRANSACTION 'CO03' AND SKIP FIRST SCREEN.
      ENDIF.

    WHEN 'ZAGRUPA'.  "Botão → ZAGRUPA
      CALL TRANSACTION 'ZAGRUPA'.

    WHEN 'ZDPR'.  "Botão → ZDPR
      CALL TRANSACTION 'ZDPR'.

  ENDCASE.
ENDFORM.
