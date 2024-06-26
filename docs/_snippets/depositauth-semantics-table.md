<table class="depauth">
<thead>
  <tr>
    <th>&nbsp;</th>
    <th colspan="2" class="depauth-status">DepositAuth Disabled</th>
    <th class="depauth-spacer">&nbsp;</th>
    <th colspan="2" class="depauth-status">DepositAuth Enabled</th>
  </tr>
  <tr>
    <th>Transaction Type</th>
    <th>Sent by Destination</th> <th>Sent by Others</th>
    <th class="depauth-spacer">&nbsp;</th>
    <th>Sent by Destination</th> <th>Sent by Others</th> <th>Sent by Preauthorized Others</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="depauth-txtype">AccountSet</td>
    <td colspan="6" class="depauth-na">(This transaction type never sends money.)</td>
  </tr>
  <tr>
    <td class="depauth-txtype">CheckCancel</td>
    <td colspan="6" class="depauth-na">(This transaction type never sends money.)</td>
  </tr>
  <tr>
    <td class="depauth-txtype">CheckCash</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-no">No Permission</td>
    <td class="depauth-spacer">&nbsp;</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-no">No Permission</td>
    <td class="depauth-no">No Permission</td>
  </tr>
  <tr>
    <td class="depauth-txtype">CheckCreate</td>
    <td colspan="6" class="depauth-na">(This transaction type never sends money.)</td>
  </tr>
  <tr>
    <td class="depauth-txtype">EscrowCancel</td>
    <td class="depauth-maybe" colspan="6">Can return XRP from an expired escrow</td>
  </tr>
  <tr>
    <td class="depauth-txtype">EscrowCreate</td>
    <td colspan="6" class="depauth-na">(This transaction type can only debit XRP, not credit it.)</td>
  </tr>
  <tr>
    <td class="depauth-txtype">EscrowFinish</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-spacer">&nbsp;</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-no">No Permission</td>
    <td class="depauth-ok">OK</td>
  </tr>
  <tr>
    <td class="depauth-txtype">OfferCancel</td>
    <td colspan="6" class="depauth-na">This transaction type never sends money.</td>
  </tr>
  <tr>
    <td class="depauth-txtype">OfferCreate</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-maybe">Only if account previously created a matching offer</td>
    <td class="depauth-spacer">&nbsp;</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-maybe">Only if account previously created a matching offer</td>
    <td class="depauth-maybe">Only if account previously created a matching offer</td>
  </tr>
  <tr>
    <td class="depauth-txtype">Payment <div class="depauth-subtype">(If account has more than the minimum XRP reserve, enables No Ripple on all trust lines, and places no offers)</div></td>
    <td class="depauth-maybe">Cross-currency only</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-spacer">&nbsp;</td>
    <td class="depauth-maybe">Cross-currency only <sup>1</sup></td>
    <td class="depauth-no">No Permission</td>
    <td class="depauth-ok">OK</td>
  </tr>
  <tr>
    <td class="depauth-txtype">Payment <div class="depauth-subtype">(If account XRP balance is below the minimum XRP reserve)</div></td>
    <td class="depauth-maybe">Cross-currency only</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-spacer">&nbsp;</td>
    <td class="depauth-maybe">Cross-currency only <sup>1</sup></td>
    <td class="depauth-maybe">XRP payments up to the minimum reserve</td>
    <td class="depauth-ok">OK</td>
  </tr>
  <tr>
    <td class="depauth-txtype">Payment <div class="depauth-subtype">(If account has any trust lines with No Ripple disabled)</div></td>
    <td class="depauth-maybe">Cross-currency only</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-spacer">&nbsp;</td>
    <td class="depauth-maybe">Cross-currency only <sup>1</sup></td>
    <td class="depauth-maybe">Balance changes from rippling</td>
    <td class="depauth-ok">OK</td>
  </tr>
  <tr>
    <td class="depauth-txtype">Payment <div class="depauth-subtype">(If account has placed offers)</div></td>
    <td class="depauth-maybe">Cross-currency only</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-spacer">&nbsp;</td>
    <td class="depauth-maybe">Cross-currency only <sup>1</sup></td>
    <td class="depauth-maybe">Balance changes from executing offers</td>
    <td class="depauth-ok">OK</td>
  </tr>
  <tr>
    <td class="depauth-txtype">PaymentChannelClaim</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-spacer">&nbsp;</td>
    <td class="depauth-ok">OK</td>
    <td class="depauth-no">No Permission</td>
    <td class="depauth-ok">OK</td>
  </tr>
  <tr>
    <td class="depauth-txtype">PaymentChannelCreate</td>
    <td colspan="6" class="depauth-na">(This transaction type can only debit XRP, not credit it.)</td>
  </tr>
  <tr>
    <td class="depauth-txtype">PaymentChannelFund</td>
    <td class="depauth-maybe" colspan="6">Can return XRP when closing a channel created by self</td>
  </tr>
  <tr>
    <td class="depauth-txtype">SetRegularKey</td>
    <td colspan="6" class="depauth-na">(This transaction type never sends money.)</td>
  </tr>
  <tr>
    <td class="depauth-txtype">SignerListSet</td>
    <td colspan="6" class="depauth-na">(This transaction type never sends money.)</td>
  </tr>
  <tr>
    <td class="depauth-txtype">TrustSet</td>
    <td colspan="6" class="depauth-na">(This transaction type never sends money.)</td>
  </tr>
</table>

<p class="depauth-subtype"><sup>1</sup>: The DepositPreauth amendment fixes a bug in DepositAuth which causes cross-currency payments to oneself to fail if the account requires deposit authorization. If the DepositPreauth amendment is not enabled, these cases result in "No Permission" instead.</p>

<style type="text/css">
.depauth-txtype {
  font-weight: bold;
}
.depauth-status {
  text-align: center;
}
.depauth-na {
  background-color: var(--gray-dark);
  color: var(--white);
}
.depauth-ok {
  background-color: var(--success);
  color: black;
}
.depauth-no {
  background-color: var(--danger);
  color: black;
}
.depauth-maybe {
  background-color: var(--warning);
  color: black;
}
.depauth-subtype {
  font-weight: normal;
  font-size: 8pt;
}
.depauth-spacer {
  background-color: var(--secondary);
  padding: 2px !important;
  font-size: 2px;
  border: 1px solid var(--secondary);
}
</style>
