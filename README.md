# api-DB-Transaction


 public function withdraw_status_approve(Request $request)
    {
//       dd($request->all());
        if ($request->ajax()) {
            $status = false;

            DB::beginTransaction();****************************************************
            try{
                $agents = AgentPaymentRequest::find($request->id);
                $agents->status = $request->status;
                $agents->transactional_note = $request->note;
                $agents->save();


                $agent_wallet_transactions = new AgentWalletTransaction();
                $agent_wallet_transactions->agent_id = $request->agent;
                $agent_wallet_transactions->credit = 0;
                $agent_wallet_transactions->transaction_id = Str::uuid();
                $agent_wallet_transactions->debit = $request->amount;
                $agent_wallet_transactions->admin_bonus = 0;
                $agent_wallet_transactions->balance = 0;
                $agent_wallet_transactions->save();

                DB::commit();
                $status = true;
            }
            catch (\Exception $e){
                DB::rollBack();
            }

            $data = $request->status;
            return response()->json($data);
        }
    }
    
    ###############################################################################################
    
    
    
    
    
    return response()->json([
                'status' => 'success',
                'message' => 'List of orders',
                'data' => [
                    'total' => $total_orders,
                    'page' => $request->post('page', 1),
                    'no_of_rows' => count($orders),
                    'data' => $orders
                ]
            ]);
9:57
public function index(Request $request)
    {
        try{
            $limit = $request->post('no_of_rows', 10);
            $offset = ($request->post('page', 1) - 1) * $limit;
            $orders = AgentAffiliate::where('agent_id', Auth::id());
            $total_orders = $orders->count();
            $orders = $orders->orderBy('id', 'desc')
                ->skip($offset)
                ->take($limit)
                ->selectRaw('
                    DATE_FORMAT(created_at,"%d-%m-%Y %h:%i %p") as order_date,
                    order_id,
                    customer_id,
                    total_products,
                    total_amount
                ')
                ->get();
            return response()->json([
                'status' => 'success',
                'message' => 'List of orders',
                'data' => [
                    'total' => $total_orders,
                    'page' => $request->post('page', 1),
                    'no_of_rows' => count($orders),
                    'data' => $orders
                ]
            ]);
        }
        catch (\Exception $e){
            return response()->json([
                'status' => 'failed',
                'message' => 'Order data couldn\'t be loaded',
                'data' => []
            ]);
        }
    }
