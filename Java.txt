package br.com.bhmedical.comissao.gerarfinanceiro;

import br.com.sankhya.extensions.actionbutton.AcaoRotinaJava;
import br.com.sankhya.extensions.actionbutton.ContextoAcao;
import br.com.sankhya.extensions.actionbutton.QueryExecutor;
import br.com.sankhya.extensions.actionbutton.Registro;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;


public class GerarFinanceiro implements AcaoRotinaJava {
	
	@Override
	public void doAction(ContextoAcao contexto) throws Exception {
			Integer codparceiro = 0;
			Double comissao = 0.0;
         Registro[] linhasSelecionadas = contexto.getLinhas();
       
 		Integer contabanc = (Integer) contexto.getParam("contabanc");
 		Integer tipotit = (Integer) contexto.getParam("tipotit");
 		String historico = (String) contexto.getParam("historico");
 		Integer natureza = (Integer) contexto.getParam("natureza");
 		Integer centroresult = (Integer) contexto.getParam("centroresult");
 		Integer banco = (Integer) contexto.getParam("banco");
 		Integer tipoper = (Integer) contexto.getParam("tipoper");
 		Integer empresa = (Integer) contexto.getParam("empresa");
 		Date dtvenc = (Date) contexto.getParam("dtvenc");
 		Date dtneg = (Date) contexto.getParam("dtneg");
         
         Map<Integer, Double> parceiros = new HashMap<>();
         Map<Integer, String> informacoesAdicionais = new HashMap<>(); //NovoHashMap
         //Map<Integer, String> informacoesAdicionaisProdutos = new HashMap<>(); //NovoHashMap
         
         for(Registro linha : linhasSelecionadas) {
        	 QueryExecutor query = contexto.getQuery();
 			query.setParam("P_CODID", linha.getCampo("CODID"));
 			
 			query.nativeSelect("SELECT CALC.CODID AS CODID, CALC.VALOR AS VALORCOMISSAO, CALC.NOTA AS NOTA, CALC.CODPARC AS CODPARCE, CALC.CODPROD AS CODPROD, CALC.NUFIN, CALC.CODGRUPOPROD,CALC.LANCADO AS LANCADO\r\n"
 					+ "FROM AD_LANCAMENTOS CALC\r\n"
 					+ "WHERE CALC.CODID = {P_CODID}");
 			
 			
 			

 			while(query.next()) {
 				Integer lancado = query.getInt("LANCADO");
 			    
 			    if (lancado.equals(1)) {
 			        contexto.mostraErro("O(s) Título(s) selecionado(s), já possuem financeiro gerado!");
 			    }
			  continue;
			      
 			}
       
         }
         
         
         
         
      for(Registro linha : linhasSelecionadas) {		
		    	
			QueryExecutor query = contexto.getQuery();
			
			query.setParam("P_CODID", linha.getCampo("CODID"));
			
			
			query.nativeSelect("SELECT CALC.CODID AS CODID, CALC.VALOR AS VALORCOMISSAO, CALC.NOTA AS NOTA, CALC.CODPARC AS CODPARCE, CALC.CODPROD AS CODPROD, CALC.NUFIN, CALC.CODGRUPOPROD\r\n"
					+ "FROM AD_LANCAMENTOS CALC\r\n"
					+ "WHERE CALC.CODID = {P_CODID}");
			

			
			while(query.next()) {
			    Integer codigodoparceiro = query.getInt("CODPARCE");
			    Double valorComissao = query.getDouble("VALORCOMISSAO");
			    //query.update("INSERT INTO AD_NUMEROS(NUMEROS) VALUES ((SELECT MAX(NUMEROS)+1 FROM AD_NUMEROS))");
				//query.update("INSERT INTO AD_TITULOSGERADOS(CODTITULOSGERADOS,CODPROD,CODGRUPOPROD,NUNOTA) VALUES ((SELECT MAX(CODTITULOSGERADOS)+1 FROM AD_TITULOSGERADOS)," + linha.getCampo("CODPROD") + "," + linha.getCampo("CODGRUPOPROD")+ ","+ linha.getCampo("NOTA")+ ")" );
			    query.update("UPDATE AD_LANCAMENTOS SET LANCADO = 1 WHERE CODID = " + linha.getCampo("CODID"));
			    parceiros.put(codigodoparceiro, parceiros.getOrDefault(codigodoparceiro, 0.0) + valorComissao);
			    Integer nota = query.getInt("CODID");
			    //Integer produto = query.getInt("CODPROD");
	            String informacaoAtual = informacoesAdicionais.getOrDefault(codigodoparceiro, "");
	            //String informacaoAtualProduto = informacoesAdicionaisProdutos.getOrDefault(codigodoparceiro, "");
	            
	            if (!informacaoAtual.contains(String.valueOf(nota))) {
	            if (!informacaoAtual.isEmpty()) {
	                informacaoAtual += ",";
	            }}
	            
	            //if (!informacaoAtualProduto.contains(String.valueOf(produto))) {
		            //if (!informacaoAtualProduto.isEmpty()) {
		            	//informacaoAtualProduto += ",";
		            //}}
	            
	            informacaoAtual += nota;
	            //informacaoAtualProduto += produto;
	            
	            informacoesAdicionais.put(codigodoparceiro, informacaoAtual);
	            //informacoesAdicionaisProdutos.put(codigodoparceiro, informacaoAtualProduto);
				}
			}
      
			      for (Map.Entry<Integer, Double> entry : parceiros.entrySet()) {
			    	
			    	  codparceiro = entry.getKey();
					  comissao = entry.getValue();
					  String informacaoAdicional = informacoesAdicionais.get(codparceiro);
					  //String informacaoAdicionalProduto = informacoesAdicionaisProdutos.get(codparceiro);
					  
					  
					  criaProvisaoFinanceira
					  (contexto, 
							  comissao, 
							  codparceiro,
							  informacaoAdicional, 
							  contabanc, 
							  tipotit, 
							  historico,
							 natureza,
							 centroresult,
							  banco,
							 tipoper,
							   empresa,
							 dtvenc,
							 dtneg
							 ); 
			      }	
     
      }
	
	public void criaProvisaoFinanceira(ContextoAcao contexto, 
			   Double valorComissao, 
			   Integer codparceiro,
			   String informacaoAdicional,
			   Integer contabanc, 
			   Integer tipotit, 
			   String historico,
			   Integer natureza,
			   Integer centroresult,
			   Integer banco,
			   Integer tipoper,
			   Integer empresa,
			   Date dtvenc,
			   Date dtneg
			   //,String informacaoAdicionalProduto
		      ) throws Exception {				
	
					Registro financeiro = contexto.novaLinha("TGFFIN");
					
					financeiro.setCampo("VLRDESDOB", Math.abs(valorComissao));
					
					financeiro.setCampo("RECDESP", valorComissao < 0 ? 1 : -1);
					
					financeiro.setCampo("PROVISAO", "N");
					
					financeiro.setCampo("CODEMP", empresa);
					
					financeiro.setCampo("CODTIPOPER", tipoper);
					
					financeiro.setCampo("NUMNOTA", 0);
					
					financeiro.setCampo("DESDOBRAMENTO", "1");
					
					financeiro.setCampo("DESDOBDUPL", "T");
					
					financeiro.setCampo("DTNEG", dtneg);
					
					financeiro.setCampo("DTVENC", dtvenc);          
					
					financeiro.setCampo("CODPARC", codparceiro);
					
					financeiro.setCampo("CODVEND", 0);
					
					financeiro.setCampo("CODNAT", natureza);
					
					financeiro.setCampo("CODCENCUS", centroresult);
					
					financeiro.setCampo("HISTORICO", historico);
					financeiro.setCampo("AD_INFORMACOES_ADICIONAIS", informacaoAdicional);
					//financeiro.setCampo("AD_CODIGOS_PRODUTOS", informacaoAdicionalProduto);
					financeiro.setCampo("CODBCO", banco);
					
					financeiro.setCampo("CODTIPTIT", tipotit);
					
					financeiro.save();
					
			        //query.update("UPDATE TGFFIN SET AD_COMISSAOGERENTE = (SELECT MAX(NUMEROS) FROM AD_NUMEROS) WHERE NUFIN="+ financeiro.getCampo("NUFIN"));
			        
			        
					}

}

