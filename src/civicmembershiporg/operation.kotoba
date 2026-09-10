(ns civicmembershiporg.operation
  "Operation flow: intake → advise → govern → decide → commit | hold | escalate.
  This is a simplified demo StateGraph-equivalent for single-turn proposals."
  (:require [civicmembershiporg.store :as store]
            [civicmembershiporg.governor :as governor]
            [civicmembershiporg.advisor :as advisor]))

;; ---------------------- state machine steps ----------------------

(defn intake-proposal
  "Intake: validate structure."
  [proposal]
  (if (and (:operation proposal) (:effect proposal))
    {:state :intake-ok :proposal proposal}
    {:state :intake-fail :reason "Missing :operation or :effect"}))

(defn advise-proposal
  "Advise: add reasoning and confidence to the proposal."
  [store proposal]
  (advisor/advise-proposal store proposal))

(defn govern-proposal
  "Govern: apply three HARD checks."
  [store proposal]
  (governor/govern store proposal))

(defn decide-proposal
  "Decide: translate governor decision + special handling for escalation ops."
  [govern-result]
  (let [op-id (:operation (:proposal govern-result))
        decision (:decision govern-result)]
    (if (not (:passes? govern-result))
      {:action :held
       :reason (str "Governor rejected: " (mapv :violation (:violations govern-result)))}
      (if (= op-id :flag-safety-concern)
        {:action :escalated
         :reason "Safety/conduct concerns always escalate for human review"}
        {:action :pending-approval
         :reason "Proposal passed governance, awaiting approval"}))))

(defn commit-proposal
  "Commit: record the decision and any state changes."
  [store proposal result]
  (store/append-ledger! store
    {:proposal proposal
     :result result
     :timestamp "2026-07-15T00:00:00Z"})
  result)

;; ---------------------- end-to-end flow ----------------------

(defn run-proposal
  "Run a single proposal through the full pipeline."
  [store proposal]
  (let [intake (intake-proposal proposal)]
    (if (= :intake-fail (:state intake))
      {:action :failed :reason (:reason intake)}
      (let [advised (advise-proposal store proposal)
            governed (govern-proposal store advised)
            decided (decide-proposal governed)]
        (commit-proposal store advised decided)
        decided))))
